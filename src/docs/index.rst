ErlPort - Documentation
=======================

.. meta::
   :keywords: erlport erlang python ruby docs documentation
   :description: Documentation for ErlPort library

ErlPort documentation
+++++++++++++++++++++

.. sidebar:: Language specific documentation

    `Python <python.html>`__
        Details and examples for Python
    `Ruby <ruby.html>`__
        Details and examples for Ruby

.. contents::

.. include:: include/erlport.rst

Please check `Downloads </downloads/>`_ page for installation instructions and
packages to download.

ErlPort Features
----------------

Easy inter-language function calls
  ErlPort supports *Module-Function-Arguments (MFA)* API to call a function in
  any supported language. This scheme also allows to create a chain of calls by
  passing another one MFA as arguments.

Easy inter-language message passing
  All external languages have an API to send or handle messages sent from other
  languages. Moreover on Erlang side all ErlPort related messages will be
  redirected to external language processes.

Support for custom data types
  All external languages support subset of `Erlang data types
  <http://www.erlang.org/doc/reference_manual/data_types.html>`__ as the common
  data types mapping. Also a special data type exists to transfer language
  specific data types between the same type of languages automatically. And
  moreover it's possible to create custom encoders/decoders to support higher
  level data types mapping.

Support for all recent versions of the languages
  ErlPort should work on any operating system (including Windows) on which
  supported languages are work. All recent versions of Erlang, Python and Ruby
  are supported. Please check `Downloads </downloads/>`__ page for more
  details.

How ErlPort works
-----------------

To be able to communicate with an external language first an instance of the
language should be created from Erlang. Usually an instance of the language can
be created with ``start/0`` or ``start/1`` functions which can be found in an
Erlang module for the specific language. During the ``start`` call ErlPort
trying to start the language specific part of ErlPort library and supply the
options passed to ``start/1`` or default options for ``start/0``.

When the language specific part of ErlPort library will be started new OS
process will be created and connected to Erlang with a `port
<http://www.erlang.org/doc/reference_manual/ports.html>`__. At this point
ErlPort functions which work with language instances can be used, for example
``call/4`` or ``cast/2`` functions.

For each call of instance related functions like ``call/4`` or ``cast/2``
a message will be created and send to the port. The language specific part of
ErlPort library on the other side of the port will handle the message by
calling the function or message handler for example. The result of the
operation (if applicable) will be passed back to the port and handled by the
ErlPort library in Erlang. The same algorithm but in the opposite direction
also works if external language want to call a function in Erlang.

At the end of the session when ``stop/1`` function will be called the port
connected to the language instance will be closed and the language specific
part of ErlPort library will finish the OS process of the instance.
