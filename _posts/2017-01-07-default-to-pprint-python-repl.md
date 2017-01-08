---
layout: post
title: "Making the Python REPL output Pretty"
description: "Making pprint the default formatter"
tags: [Python, REPL]
comments: true
share: true
---


Recently, there was a [tweet by Ned Batchelder](https://twitter.com/nedbat/status/817827164443840512 )
that illustrated how to make Python REPL output prettier. I went to implement it,
and wanted to put together some instructions for the future.

## PYTHONSTARTUP

The PYTHONSTARTUP referenced here is an environment variable that points to a python
file that we can place a series of command into that will be evaluated when we
launch a Python REPL. I have the following code in a file called
`.python-startup.py`.

    {% highlight python %}
    import pprint
    import sys

    sys.ps1 = "\033[0;34m>>> \033[0m"
    sys.ps2 = "\033[1;34m... \033[0m"
    sys.displayhook = pprint.pprint
    {% endhighlight %}

The code above will set the `>>>` to a light blue and the `...` to a darker blue,
but this isn't the part you are here for probably. You want the next line, which
sets the displayhook for output to pretty print.

Next, you can export the PYTHONSTARTUP environment variable pointing to your
file as shown here.

    export PYTHONSTARTUP=~/.python-startup.py

You can also add this to your `.bashrc` or `.zshrc` depending on which shell you
are using, and it will always make your python repl output be pretty.

## So what is the difference?

First let's look at the normal output:

    {% highlight shell %}
    >>> dessert = {'cookies': {'chocolate chip': 1, 'oatmeal raisin': 12, 'peanut butter': 3},
    ...     'cake': 'OMG NO!',
    ...     'pie': {'apple': 1, 'peach': 2, 'fudge': 0}}
    >>> dessert
    {'cookies': {'chocolate chip': 1, 'oatmeal raisin': 12, 'peanut butter': 3}, 'cake': 'OMG NO!', 'pie': {'apple': 1, 'peach': 2, 'fudge': 0}}
    {% endhighlight %}

Now let's see it with the pretty print in place:

    {% highlight shell %}
    >>> dessert = {'cookies': {'chocolate chip': 1, 'oatmeal raisin': 12, 'peanut butter': 3},
    ...     'cake': 'OMG NO!',
    ...     'pie': {'apple': 1, 'peach': 2, 'fudge': 0}}
    >>> dessert
    {'cake': 'OMG NO!',
     'cookies': {'chocolate chip': 1, 'oatmeal raisin': 12, 'peanut butter': 3},
      'pie': {'apple': 1, 'fudge': 0, 'peach': 2}}
    {% endhighlight %}

If your are curious what the colors look like:
![Output Preview]({{ site.url }}/assets/python-prompt-example.png)
