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

    error:{python, ExceptionClass, ExceptionArgument, ReversedStackTrace}

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
    1>                         {python, "python3"}]).
    {ok,<0.34.0>}
    2> python:call(P, version, version, []).
    "3.2.3 (default, Oct 19 2012, 20:10:41) \n[GCC 4.6.3]"
    3> python:stop(P).
    ok

Call Erlang functions from Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: Python call API

    Check `Python API`_ and `data types mapping`_ sections for more details
    about `erlport.erlang.call()`_ function and supported data types.

ErlPort uses Python `erlport.erlang` module as an interface to Erlang. Namely
`erlport.erlang.call()`_ function allows to call Erlang functions from Python.
The `erlport.erlang.call()`_ function accepts Module and Function arguments as
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

And here is the table of Python data types to Erlang data types mapping. The
types mapping between Erlang and Python are practically orthogonal:

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
| *Other Python data type*             | *Opaque Python data type container*  |
+--------------------------------------+--------------------------------------+
| *Opaque data type container*         | *Opaque data type container*         |
+--------------------------------------+--------------------------------------+

.. _erlport.erlterms.Atom():

erlport.erlterms.Atom(string)
    Class to represents Erlang atoms in Python. The ``string`` argument should
    be a byte string (str() in Python 2 or bytes() in Python 3) not longer that
    255 bytes. Each ``Atom`` instance is a singleton the same as in Erlang.

.. _erlport.erlterms.List():

erlport.erlterms.List(list)
    Class to represents Erlang lists in Python. Basically just a subclass of
    list() with the following one additional method:

    List.to_string()
        Convert list content to an Unicode string (unicode() in Python 2 or
        str() in Python 3). There's no distinct string data type in Erlang so
        lists should be explicitly converted to strings with this method.

.. _erlport.erlterms.ImproperList():

erlport.erlterms.ImproperList(list, tail)
    Class to represents Erlang improper lists in Python. The ``tail`` argument
    can't be a list. *Note that this class exists mostly to convert improper
    lists received from Erlang side and probably there are no reasons to create
    instances of this class in Python.*

.. _Erlang functions:

Erlang API
~~~~~~~~~~

.. _python:start/0:

python:start() -> {ok, Pid} | {error, Reason}
    Start Python instance with the default options

.. _python:start/1:

python:start(Options) -> {ok, Pid} | {error, Reason}
    Start Python instance with options. The ``Options`` argument should be
    a list with the following options.

    General options:

    {buffer_size, Size::pos_integer()}
        Size in bytes of the ErlPort receive buffer on Python side. The default
        is 65536 bytes.
    {call_timeout, Timeout::pos_integer() | infinity}
        Default timeout in milliseconds for function calls. Per call timeouts
        can be set with `python:call/5`_ function.
    {cd, Path::string()}
        Change current directory to ``Path`` before starting.
    {compressed, 0..9}
        Set terms compression level. `0` means no compression and `9` will take
        the most time and *may (or may not)* produce a smaller result. Can be
        used as an optimisation if you know that your data can be easily
        compressed.
    {env, [{Name::string(), Value::string() | false}]}
        Set environment for Python instance. The ``Name`` variable is the name
        of environment variable to set and ``Value`` can be a string value of
        the environment variable or ``false`` if the variable should be
        removed.
    nouse_stdio
        Not use `STDIN/STDOUT <http://en.wikipedia.org/wiki/Standard_streams>`_
        for communication. *Not supported on Windows.*
    {packet, 1 | 2 | 4}
        How many bytes to use for the packet size. The default is 4 which means
        that packets can be as big as 4GB but if you know that your data will
        be small you can set it for example to 1 which limits the packet size
        to 256 bytes but also saves 3 bytes for each packet. *Note however that
        ErlPort adds some meta-information in each packet so the resulting
        packets always will be bigger than your expected size.*
    {start_timeout, Timeout::pos_integer() | infinity}
        Time to wait for the instance to start.
    use_stdio
        Use `STDIN/STDOUT <http://en.wikipedia.org/wiki/Standard_streams>`_ for
        communication. The default.

    Python related options:

    {python, Python::string()}
        Path to the Python interpreter executable
    {python_path, Path::string() | [Path::string()]}
        The Python modules search path. The ``Path`` variable can be a string
        in `PYTHONPATH
        <http://docs.python.org/2/using/cmdline.html#envvar-PYTHONPATH>`_
        format or a list of paths. The priorities of different ways to set the
        modules search path is as follows:

        #. *python_path* option
        #. *PYTHONPATH* environment variable set through the *env* option
        #. *PYTHONPATH* environment variable

.. _python:start/2:

python:start(Name, Options) -> {ok, Pid} | {error, Reason}
    Start named Python instance. The instance will be registered with ``Name``
    name. The ``Options`` variable is the same as for `python:start/1`_.

.. _python:start_link/0:

python:start_link() -> {ok, Pid} | {error, Reason}
    The same as `python:start/0`_ except the link to the current process is
    also created.

.. _python:start_link/1:

python:start_link(Options) -> {ok, Pid} | {error, Reason}
    The same as `python:start/1`_ except the link to the current process is
    also created.

.. _python:start_link/2:

python:start_link(Name, Options) -> {ok, Pid} | {error, Reason}
    The same as `python:start/2`_ except the link to the current process is
    also created.

.. _python:stop/1:

python:stop(Instance) -> ok
    Stop Python instance

.. _python:call/4:

python:call(Instance, Module, Function, Arguments) -> Result
    Call Python function. The ``Instance`` variable can be a pid() which
    returned by one of the ``python:start`` functions or an instance name
    (atom()) if the instance was registered with a name. The ``Module`` and
    ``Function`` variables should be atoms and ``Arguments`` is a list.

    In case of any error on Python side during the function call an exception
    of class `error <http://www.erlang.org/doc/reference_manual/errors.html>`_
    will be generated in the following form:

    .. sourcecode:: erlang

        error:{python, ExceptionClass, ExceptionArgument, ReversedStackTrace}

.. _python:call/5:

python:call(Instance, Module, Function, Arguments, Options) -> Result
    The same as `python:call/4`_ except the following options can be added:

    {timeout, Timeout::pos_integer() | infinity}
        Call timeout in milliseconds.

.. _python:cast/2:

python:cast(Instance, Message) -> ok
    Send a message to the python instance.

.. _Python functions:

Python API
~~~~~~~~~~

.. _erlport.erlang.call():

erlport.erlang.call(module, function, arguments) -> result
    Call Erlang function as ``module:function(arguments)``. The
    ``function`` and ``module`` variables should be of type
    `erlport.erlterms.Atom()`_ and ``arguments`` should be a list.

.. _erlport.erlang.cast():

erlport.erlang.cast(pid, message)
    Send a message to Erlang. The ``pid`` and ``message`` variables should be
    the same types as supported by `Erlang ! (send) expression
    <http://www.erlang.org/doc/reference_manual/expressions.html#id77156>`_.
    Erlang ``pid()`` variables however can't be created in Python but can be
    passed as parameters from Erlang.

.. _erlport.erlang.self():

erlport.erlang.self() -> pid
    Get the Erlang pid of the Python instance

.. _erlport.erlang.set_encoder():

erlport.erlang.set_encoder(encoder)
    Set encoder for custom data types

.. _erlport.erlang.set_decoder():

erlport.erlang.set_decoder(decoder)
    Set decoder for custom data types

.. _erlport.erlang.set_message_handler():

erlport.erlang.set_message_handler(handler)
    Set message handler

.. _erlport.erlang.set_default_encoder():

erlport.erlang.set_default_encoder()
    Reset custom data types encoder to the default which just pass the term
    through without any modifications

.. _erlport.erlang.set_default_decoder():

erlport.erlang.set_default_decoder()
    Reset custom data types decoder to the default which just pass the term
    through without any modifications

.. _erlport.erlang.set_default_message_handler():

erlport.erlang.set_default_message_handler()
    Reset message handler to the default which just ignores all the incoming
    messages

.. _environment variables:

Environment variables
~~~~~~~~~~~~~~~~~~~~~

The following environment variables can change the default behavior of
ErlPort:

ERLPORT_PYTHON
    Path to Python interpreter executable which will be used by default.

PYTHONPATH
    The default search patch for module files. The same as `PYTHONPATH
    <http://docs.python.org/2/using/cmdline.html#envvar-PYTHONPATH>`_
    environment variable supported by Python. The priorities of different
    ways to set the modules search path is as follows:

    #. *python_path* option
    #. *PYTHONPATH* environment variable set through the *env* option
    #. *PYTHONPATH* environment variable
