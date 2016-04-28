---
layout: post
title: "Parallelizing Queries with SQLAlchemy, Gevent, and PostgreSQL"
description: "Adventures in read old forum posts."
tags: [SQLAlchemy, Postgresql, Postgres, Gevent]
comments: true
share: true
---

Recently, we had a need to execute multiple SQLAlchemy queries in a parallel fashion against a PostgreSQL database with Python 2 and [psycopg2](http://initd.org/psycopg/). We didn't really need a full scale multithreading approach, so we turned to [gevent](http://www.gevent.org/index.html). Gevent is a implementation of [green threading](https://en.m.wikipedia.org/wiki/Green_threads) that uses libevent. This post really isn't about what threading model is better, as that is great subjective and changes based on your use case and needs.

## Making Psycopg2 Coroutine Friendly

So the first thing we need to do to prepare to leverage gevent is make sure psycopg2 is configured properly.  Because the main psycopg2 is a C extension, we can not just take advantage of gevent's monkey patching. Nonetheless, psycopg2 exposes a [hook](http://initd.org/psycopg/docs/extensions.html#psycopg2.extensions.set_wait_callback) that can be use with coroutine libraries to integrate with the event scheduler. There are a few libraries to help with this such as [sqlalchemy_gevent](https://pypi.python.org/pypi/sqlalchemy-gevent) or [psycogreen](https://bitbucket.org/dvarrazzo/psycogreen) that you can use to automatically set this part up for you. However, the code is fairly short and straight forward. Let's look at the code from psycogreen, that I use in my applications.

    {% highlight python %}
    import psycopg2
    from psycopg2 import extensions

    from gevent.socket import wait_read, wait_write

    def make_psycopg_green():
        """Configure Psycopg to be used with gevent in non-blocking way."""
        if not hasattr(extensions, 'set_wait_callback'):
            raise ImportError(
                "support for coroutines not available in this Psycopg version (%s)"
                % psycopg2.__version__)

        extensions.set_wait_callback(gevent_wait_callback)

    def gevent_wait_callback(conn, timeout=None):
        """A wait callback useful to allow gevent to work with Psycopg."""
        while 1:
            state = conn.poll()
            if state == extensions.POLL_OK:
                break
            elif state == extensions.POLL_READ:
                wait_read(conn.fileno(), timeout=timeout)
            elif state == extensions.POLL_WRITE:
                wait_write(conn.fileno(), timeout=timeout)
            else:
                raise psycopg2.OperationalError(
                    "Bad result from poll: %r" % state)
    {% endhighlight %}

The code above will block the calling thread with a callback if we are busy reading or writing, which will return control back to the event loop that could start working on another thread if one needs to be worked on. The wait_read function blocks the current green let until the connection is ready to read from, and the wait_write does the same thing for writing to the connection.

With psycopg2 ready to work with gevent, we can start writing the code to execute our queries. I'm going to use a few different elements from gevent to control how the queries are executed, how many queries can be run at once, and how the query results are handled.  There are many ways to use gevent and workers to accomplish this tasks, and some are quite simpler. My structure is set for future options and growth. I call this structure a QueryPool.

## Building our Gevent based QueryPool
The QueryPool consists of an input queue to hold the queries we want to run, an output queue to hold the results,  an overseer to load up the input queue, workers to actually get the queries and run them, and finally a function to drive the process.

The central element of our QueryPool, is the input queue. It is a special kind of FIFO queue called a `JoinableQueue`, which has a `.join()` method that blocks until all items in the queue have been acknowledged as processed with a `task_done()`. The input queue is loaded by an overseer as the first step of running the QueryPool. Then workers grab tasks one at a time from the input queue and when they finish their work they use the `task_done()` method to inform the queue that the item has been processed.

Let's look at the `__init__()` for our Query pool:

    {% highlight python %}
    import gevent
    from gevent.queue import JoinableQueue, Queue
    class QueryPool(object):
        def __init__(self, queries, pool_size=5):
            self.queries = queries
            self.POOL_MAX = pool_size
            self.tasks = JoinableQueue()
            self.output_queue = Queue()
    {% endhighlight %}

So in our `__init__()` method, we start by storing the queries. Next we set a POOL_MAX which is maximum number of workers we are going to use. This controls how many queries can be executed in parallel in our QueryPool. Then we setup the tasks queue, which is where are workers will pick up work from. Finally we setup the output queue that workers will publish the query results.

### Defining worker methods
Now we can look at the workers, I use two methods in the QueryPool calls. The first one contains the logic to prepare a database connection and execute the SQLAlchemy query.

    {% highlight python %}
    def __query(self, query):
        conn = engine.connect()
        results = conn.execute(query).fetchall()
    {% endhighlight %}

For the workers method within the QueryPool class, we start a loop that is going to run until their are not tasks left in the tasks (input) queue. It will get one task, call the `__query()` method with the details from the task, handle any exceptions, add the results to the output queue, and then mark the task as done. When adding the results to the output queue, we use the `put_nowait()` method of gevent queues to add the results without blocking. This allows the event loop to look for the next thing to work on.

    {% highlight python %}
    def executor(self, number):
        while not self.tasks.empty():
            query = self.tasks.get()
            try:
                results = self.__query(query)
                self.output_queue.put_nowait(results)
            except Exception as exc_info:
                print exc_info
                print 'Query failed :('
            self.tasks.task_done()
    {% endhighlight %}

### Building an overseer method
Now that we have a something to handle the work, we need to load the task (input) queue. I use an overseer method that iterates over the supplied recipes and puts them on the task (input) queue.

    {% highlight python %}
    def overseer(self):
        for query in self.queries:
            self.tasks.put_nowait(query)
    {% endhighlight %}

### Building the run method

Finally, we are ready to build the run method that ties everything together and makes it work. Let's look at the code first and then cover it step by step.

    {% highlight python %}
    def run(self):
        self.running = []
        gevent.spawn(self.overseer).join()
        for i in range(self.POOL_MAX):
            runner = gevent.spawn(self.executor, i)
            runner.start()
            self.running.append(runner)

        self.tasks.join()
        for runner in self.running:
            runner.kill()
      {% endhighlight %}

The runner first spawn the overseer and immediately calls join on it, which blocks until the overseer is done loading items into the tasks (input) queue. This is overkill, but provides more options and would allow up to load asynchronously in the future. Next, we start a number of workers up to the POOL_MAX we defined earlier, and add them to a list so we can control them later. When we start them, they immediately begin working. Then we call `.join()` on the tasks (input) queue which will block execution until all the queries in the queue has been execute and acknowledge as completed. Finally, we clean up by using the `.kill()` method on all of the workers.

## Using the QueryPool

Now we are ready to use our QueryPool class! I've created an Jupyter notebook available on [GitHub](https://github.com/jasonamyers/gevent-sqlalchemy/blob/master/Gevent%20SQLAlchemy.ipynb) with some example queries against a benchmarking database. I created my test database using the pgbench command to create a 5Gb database.

### Execution Results

I setup 6 queries of varying complexity and execution time. I then executed those queries in a traditional serial execution style and then with our QueryPool using different worker counts.

* Serial: 24 seconds
* QueryPool - 5 workers: 8 seconds
* QueryPool - 3 workers: 9 seconds
* QueryPool - 2 workers: 15 seconds
