ErlPort - Documentation
=======================

.. meta::
   :keywords: erlport erlang python ruby docs documentation
   :description: Documentation for ErlPort library

ErlPort documentation
+++++++++++++++++++++

.. contents::

.. include:: include/erlport.rst

Features
--------

.. include:: include/features.rst

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
