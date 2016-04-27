---
layout: post
title: "Python 2.X ImportError with PyEnv"
description: "A very annoying problem with some PyPI packages and Pyenv"
tags: [pyenv, error]
comments: true
share: true
---


If you use [pyenv](https://github.com/yyuu/pyenv/), and have installed or updated Pyenv and built Python 2.x after Feb 17 or rolled your own Python 2.x by hand, you may experience an issue where some packages will fail to run with an error message like the following: `Traceback (most recent call last): File "/Users/jasonmyers/.virtualenvs/test2/bin/jupyter-notebook", line 7, in <module>from notebook.notebookapp import main File "/Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/notebook/notebookapp.py", line 32, in <module>from zmq.eventloop import ioloop File "/Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/__init__.py", line 66, in <module>from zmq import backend File "/Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/backend/__init__.py", line 40, in <module>reraise(*exc_info) File "/Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/backend/__init__.py", line 27, in <module>_ns = select_backend(first) File "/Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/backend/select.py", line 27, in select_backend mod = __import__(name, fromlist=public_api) File "/Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/backend/cython/__init__.py", line 6, in <module>from . import (constants, error, message, context, ImportError: dlopen(/Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/backend/cython/constants.so, 2): Symbol not found: _PyUnicodeUCS2_DecodeUTF8 Referenced from: /Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/backend/cython/constants.so Expected in: flat namespace in /Users/jasonmyers/.virtualenvs/test2/lib/python2.7/site-packages/zmq/backend/cython/constants.so `

This is due to the fact that recently python wheel packages switched to "wide" unicode (ucs4) via [PEP-513](https://www.python.org/dev/peps/pep-0513/). Since not all of the wheel packages have been updated, you can end up with package that use "narrow" unicode (ucs2). I've seen this in jupyter, ipython, Pillow, and other Python packages. To fix this you need to install the packages without using the wheels until they release an update. For example: `pip install --no-use-wheel jupyter` 
This will take much longer to do the install; however, jupyter will work properly after being installed this way. More details at [https://github.com/yyuu/pyenv/issues/257](https://github.com/yyuu/pyenv/issues/257).<Paste>
