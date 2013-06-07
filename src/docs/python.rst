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

    Check `Erlang API`_ section for more details about `python:start/0`_ and
    `python:start/1`_ functions.

If you read main `Documentation </docs/>`__ page you should know that before
any use of ErlPort you need to start a Python `instance
</docs/#how-erlport-works>`__. A Python instance is basically an operating
system process which represented on Erlang side as an Erlang process.

To start an instance of Python one of the variants of ``python:start`` or
``python:start_link`` functions should be used. The `python:start/0`_ function
will start Python instance with the default parameters.

To stop the running Python instance `python:stop/1`_ function should be used
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

    Check `Erlang API`_ and `data types mapping`_ sections for more details
    about `python:call/4`_ function and supported data types.

To call a function in Python `python:call/4`_ function should be used which
accepts the instance id and Module, Function and Arguments for Python function:

.. sourcecode:: erl

    1> {ok, P} = python:start().
    {ok,<0.34.0>}
    2> python:call(P, operator, add, [2, 2]).
    4

Of course in Python you can use not only modules but also packages and
imported objects can use attributes as functions so `python:call/4`_ also
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

If you want to call a function from your own Python module in most cases you
need to set the `Python path
<http://docs.python.org/2/using/cmdline.html#envvar-PYTHONPATH>`_. You can do
it with `python:start/1`_ function or *PYTHONPATH* `environment variable
<#environment-variables>`_. The `python:start/1`_ also can be used to change
the default Python interpreter. For example let's create a simple Python
module in ``/path/to/my/modules/version.py`` file:

.. sourcecode:: python

    import sys

    def version():
        return sys.version

Now we can set path to this module in `python:start/1`_ like this:

.. sourcecode:: erl

    1> {ok, P} = python:start([{python_path, "/path/to/my/modules"},
    1> {python, "python3"}]).
    {ok,<0.34.0>}
    2> python:call(P, version, version, []).
    "3.2.3 (default, Oct 19 2012, 20:10:41) \n[GCC 4.6.3]"
    3> python:stop(P).
    ok

Call Erlang functions from Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: Python call API

    Check `Python API`_ and `data types mapping`_ sections for more details
    about `erlport.erlang.call`_ function and supported data types.

ErlPort uses Python `erlport.erlang` module as an interface to Erlang. Namely
`erlport.erlang.call`_ function allows to call Erlang functions from Python.
The `erlport.erlang.call`_ function accepts Module and Function arguments as
`erlport.erlterms.Atom()`_ object and Arguments as a list. Currently each
Erlang function will be called in a new Erlang process. Let's create the
following Python module in ``processes.py`` file in the current directory which
will be added to Python path automatically by Python:

.. sourcecode:: python

    from erlport.erlterms import Atom
    from erlport.erlang import call

    def processes():
        Pid1 = call(Atom("erlang"), Atom("self"), [])
        Pid2 = call(Atom("erlang"), Atom("self"), [])
        return [Pid1, Pid2]

Now we can call this function from Erlang:

.. sourcecode:: erl

    1> {ok, P} = python:start().
    {ok,<0.34.0>}
    2> python:call(P, processes, processes, []).
    [<0.36.0>,<0.37.0>]

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

Here you can find complete description of `data types mapping`_, `Erlang
functions`_, `Python functions`_ and `environment variables`_ supported by
ErlPort.

.. _data types mapping:

Data types mapping
~~~~~~~~~~~~~~~~~~

The following table defines mapping of Erlang data types to Python data types:

+--------------------------------------+--------------------------------------+
| Erlang data type                     | Python data type                     |
+======================================+======================================+
| integer()                            | int()                                |
+--------------------------------------+--------------------------------------+
| float()                              | float()                              |
+--------------------------------------+--------------------------------------+
| atom()                               | `erlport.erlterms.Atom()`_           |
+--------------------------------------+--------------------------------------+
| binary()                             | str() in Python 2,                   |
|                                      | bytes() in Python 3                  |
+--------------------------------------+--------------------------------------+
| tuple()                              | tuple()                              |
+--------------------------------------+--------------------------------------+
| list()                               | `erlport.erlterms.List()`_           |
+--------------------------------------+--------------------------------------+
| improper_list()                      | `erlport.erlterms.ImproperList()`_   |
+--------------------------------------+--------------------------------------+
| *Opaque Python data type container*  | *Python data type*                   |
+--------------------------------------+--------------------------------------+
| *Opaque data type container*         | *Opaque data type container*         |
+--------------------------------------+--------------------------------------+

And here is the table of Python data types to Erlang data types mapping. As you
can see the types mapping between Erlang and Python are practically orthogonal:

+--------------------------------------+--------------------------------------+
| Python data type                     | Erlang data type                     |
+======================================+======================================+
| int()                                | integer()                            |
+--------------------------------------+--------------------------------------+
| float()                              | float()                              |
+--------------------------------------+--------------------------------------+
| `erlport.erlterms.Atom()`_           | atom()                               |
+--------------------------------------+--------------------------------------+
| str() in Python 2,                   | binary()                             |
| bytes() in Python 3                  |                                      |
+--------------------------------------+--------------------------------------+
| tuple()                              | tuple()                              |
+--------------------------------------+--------------------------------------+
| `erlport.erlterms.List()`_,          | list()                               |
| list(),                              |                                      |
| unicode() in Python 2,               |                                      |
| str() in Python 3                    |                                      |
+--------------------------------------+--------------------------------------+
| `erlport.erlterms.ImproperList()`_   | improper_list()                      |
+--------------------------------------+--------------------------------------+
| *Python data type*                   | *Opaque Python data type container*  |
+--------------------------------------+--------------------------------------+
| *Opaque data type container*         | *Opaque data type container*         |
+--------------------------------------+--------------------------------------+

.. _erlport.erlterms.Atom():

erlport.erlterms.Atom()
    Atom class in Python

.. _erlport.erlterms.List():

erlport.erlterms.List()
    List class in Python

.. _erlport.erlterms.ImproperList():

erlport.erlterms.ImproperList()
    Improper list class in Python

.. _Erlang functions:

Erlang API
~~~~~~~~~~

.. _python:start/0:

python:start()
    Start Python instance

.. _python:start/1:

python:start(Options)
    Start Python instance

.. _python:start/2:

python:start(Name, Options)
    Start Python instance

.. _python:start_link/0:

python:start_link()
    Start linked Python instance

.. _python:start_link/1:

python:start_link(Options)
    Start linked Python instance

.. _python:start_link/2:

python:start_link(Name, Options)
    Start linked Python instance

.. _python:stop/1:

python:stop(Instance)
    Stop Python instance

.. _python:call/4:

python:call(Instance, Module, Function, Arguments)
    Call Python function

.. _python:cast/2:

python:cast(Instance, Message)
    Send a message

.. _Python functions:

Python API
~~~~~~~~~~

.. _erlport.erlang.call:

erlport.erlang.call()
    Call Erlang function

.. _erlport.erlang.cast:

erlport.erlang.cast()
    Send a message to Erlang

.. _erlport.erlang.self:

erlport.erlang.self()
    Get the pid of Python instance

.. _erlport.erlang.make_ref:

erlport.erlang.make_ref()
    Create new Erlang reference

.. _erlport.erlang.set_encoder:

erlport.erlang.set_encoder()
    Set encoder for custom data types

.. _erlport.erlang.set_decoder:

erlport.erlang.set_decoder()
    Set decoder for custom data types

.. _erlport.erlang.set_message_handler:

erlport.erlang.set_message_handler()
    Set message handler

.. _erlport.erlang.set_default_encoder:

erlport.erlang.set_default_encoder()
    Reset encoder for custom data types

.. _erlport.erlang.set_default_decoder:

erlport.erlang.set_default_decoder()
    Reset decoder for custom data types

.. _erlport.erlang.set_default_message_handler:

erlport.erlang.set_default_message_handler()
    Reset message handler

.. _environment variables:

Environment variables
~~~~~~~~~~~~~~~~~~~~~

PYTHON
    Path to Python interpreter

PYTHONPATH
    Python path
