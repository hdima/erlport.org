ErlPort - Python documentation
==============================

.. meta::
   :keywords: erlport erlang python documentation
   :description: Documentation for Python related part of ErlPort library

ErlPort Python documentation
++++++++++++++++++++++++++++

.. contents::

Overview and examples
---------------------

In the first section of the manual you can find an overview of all related to
Python features of ErlPort and a lot of examples mostly with using of `Erlang
shell <http://www.erlang.org/doc/man/shell.html>`__. Make sure you read
`Downloads </downloads/>`__ page and installed ErlPort or know how to start
Erlang shell with ErlPort from the sources directory.

Python instances
~~~~~~~~~~~~~~~~

.. sidebar:: Erlang start/stop API

    Check `Erlang API <#erlang-api>`__ section for more details about
    ``python:start`` and ``python:stop/1`` functions.

If you read main `Documentation </docs/>`__ page you should know that before
any use of ErlPort you need to start a Python `instance
</docs/#how-erlport-works>`__. A Python instance is basically an operating
system process which represented on Erlang side as an Erlang process.

To start an instance of Python one of the variants of ``python:start`` or
``python:start_link`` functions should be used. The ``python:start/0`` function
will start Python instance with the default parameters.

To stop the running Python instance `python:stop/1` function should be used
with the instance id as the parameter.

And of course you can run more than one instance of Python, but be careful to
not create way too many instances because they can waste all the OS resources:

.. sourcecode:: erl

    1> {ok, PythonInstance1} = python:start().
    {ok,<0.34.0>}
    2> {ok, PythonInstance2} = python:start().
    {ok,<0.36.0>}
    3> python:stop(PythonInstance1)
    ok
    4> python:stop(PythonInstance2)
    ok

Call Python functions from Erlang
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: Erlang call API

    Check `Erlang API <#erlang-api>`__ and `Data types mapping
    <#data-types-mapping>`__ sections for more details about ``python:call/4``
    function and supported data types.

To call a function in Python `python:call/4` function should be used which
accepts the instance id and Module, Function and Frguments for Python function:

.. sourcecode:: erl

    1> {ok, P} = python:start().
    {ok,<0.34.0>}
    2> python:call(P, operator, add, [2, 2]).
    4

Of course in Python you can use not only modules but also packages and
imported objects can use attributes as functions so ``python:call/4`` also
support dot-separated names for Module and Function arguments:

.. sourcecode:: erl

    3> python:call(P, 'os.path', splitext, [<<"name.ext">>]).
    {<<"name">>,<<".ext">>}
    4> python:call(P, sys, 'version.__str__', []).
    <<"2.7.3 (default, Aug  1 2012, 05:14:39) \n[GCC 4.6.3]">>

In case of any error during the function call an exception of class `error
<http://www.erlang.org/doc/reference_manual/errors.html>`_ will be generated in
the following form:

.. sourcecode:: erlang

    {python, ExceptionClass, ExceptionArgument, ReversedStackTrace}

For example:

.. sourcecode:: erl

    5> try python:call(P, unknown, unknown, [])
    5> catch error:{python, Class, Argument, StackTrace} -> error
    5> end.
    error
    6> Class.
    'exceptions.ImportError'
    7> Argument.
    "No module named unknown"
    8> StackTrace.
    [{<<"/.../erlport/priv/python2/erlport/erlang.py">>,
      237,<<"_incoming_call">>,
      <<"f = __import__(module, {}, {}, [objects[0]])">>},
     {<<"/.../erlport/priv/python2/erlport/erlang.py">>,
      245,<<"_call_with_error_handler">>,<<"function(\*args)">>}]

And of course don't forget to stop the instance at the end:

.. sourcecode:: erl

    9> python:stop(P).
    ok

Call Erlang functions from Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Send messages from Erlang to Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Send messages from Python to Erlang
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Custom datatypes
~~~~~~~~~~~~~~~~

Standard output redirection
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Reference manual
----------------

Data types mapping
~~~~~~~~~~~~~~~~~~

TODO: All data types except *opaque* are orthogonal - can be created and
meaningful on both Erlang and Python sides.

Erlang API
~~~~~~~~~~

python:start/0
    Start Python instance

python:start/1
    Start Python instance

Python API
~~~~~~~~~~

Environment variables
~~~~~~~~~~~~~~~~~~~~~
