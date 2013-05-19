ErlPort - Documentation
=======================

.. contents::

ErlPort is a library for `Erlang <http://erlang.org>`__ which helps connect
Erlang and a number of other programming languages. Currently supported
external languages are `Python <python.html>`__ and `Ruby <ruby.html>`__. The
library use `Erlang external term format
<http://erlang.org/doc/apps/erts/erl_ext_dist.html>`__ and `Erlang port
protocol <http://erlang.org/doc/man/erlang.html#open_port-2>`__ to simplify
connection between the languages.

Features
--------

Installation
------------

TODO: Maybe we only need a link to downloads/?

How ErlPort works
-----------------

To be able to communicate with an external language first an instance of
a language should be started from Erlang. Usually an instance of a language can
be started with ``start/0`` or ``start/1`` functions which can be found in the
Erlang module for the specific language. During the ``start`` call ErlPort
trying to start the part of ErlPort library for the specific language and
supply the options passed to ``start/1`` (or default options for ``start/0``).

When the language specific part of ErlPort library will be started new OS
process will be created and connected with Erlang with a `port
<http://erlang.org/doc/man/erlang.html#open_port-2>`__. At this point ErlPort
functions which use the language instance can be used for example ``call/4`` or
``cast/2``.

**TODO**: How commands are passed from Erlang to the language instance and back

At the end of the session when ``stop/1`` function will be called it closes the
port connected to the language instance and the language specific part of
ErlPort library will finish the OS process of the instance.

Interfaces
----------

- `Python <python.html>`__ - documentation for ErlPort interface for Python
- `Ruby <ruby.html>`__ - documentation for ErlPort interface for Ruby

Supported versions
------------------

ErlPort should work on any Unix-like or Windows operating system on which
Erlang, Python or Ruby can work.

The following versions of the languages are supported:

- `Erlang <http://erlang.org>`__ - any Erlang version starting from R13B
- `Python <http://python.org>`__ - any Python version starting from 2.5
- `Ruby <http://ruby.org>`__ - any Ruby version starting from 1.8
