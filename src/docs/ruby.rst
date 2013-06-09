ErlPort - Ruby documentation
============================

.. meta::
   :keywords: erlport erlang ruby documentation
   :description: Documentation for Ruby related part of ErlPort library

ErlPort Ruby documentation
++++++++++++++++++++++++++

.. contents::

Overview and examples
---------------------

In the first section of the manual you can find an overview of all related to
Ruby features of ErlPort and a lot of examples mostly with using of `Erlang
shell <http://www.erlang.org/doc/man/shell.html>`__. Make sure you read
`Downloads </downloads/>`__ page and installed ErlPort or know how to start
Erlang shell with ErlPort from the sources directory.

Ruby instances
~~~~~~~~~~~~~~

.. sidebar:: Erlang start/stop API

    Check `Erlang API`_ section for more details about `ruby:start/0`_ and
    `ruby:start/1`_ functions.

If you read main `Documentation </docs/>`__ page you should know that before
any use of ErlPort you need to start a Ruby `instance
</docs/#how-erlport-works>`__. A Ruby instance is basically an operating system
process which represented in Erlang by Erlang process.

To start an instance of Ruby one of the variants of ``ruby:start`` or
``ruby:start_link`` functions should be used. The `ruby:start/0`_ function will
start Ruby instance with the default parameters.

To stop the running Ruby instance `ruby:stop/1`_ function should be used with
the instance id as the parameter.

And of course you can run more than one Ruby instance, but be careful to not
create way too many because they can waste all the OS resources.

The following is an example of start/stop API:

.. sourcecode:: erl

    1> {ok, RubyInstance1} = ruby:start().
    {ok,<0.34.0>}
    2> {ok, RubyInstance2} = ruby:start().
    {ok,<0.36.0>}
    3> ruby:stop(RubyInstance1)
    ok
    4> ruby:stop(RubyInstance2)
    ok

Call Ruby functions from Erlang
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: Erlang call API

    Check `Erlang API`_ and `data types mapping`_ sections for more details
    about `ruby:call/4`_ function and supported data types.

To call a function in Ruby `ruby:call/4`_ function should be used which
accepts the instance id and File, Function and Arguments for Ruby function:

.. sourcecode:: erl

    1> {ok, R} = ruby:start().
    {ok,<0.34.0>}
    2> ruby:call(R, '', 'Integer', [<<"2">>]).
    2

Of course in Ruby you can also have hierarchies of files and modules so
`ruby:call/4`_ supports ``/``-separated names for File and ``::``-separated
for Function arguments:

.. sourcecode:: erl

    3> ruby:call(R, 'rss/atom', 'RSS::Atom::Feed::new::version', []).
    <<"1.0">>

In case of any error during the function call an exception of class `error
<http://www.erlang.org/doc/reference_manual/errors.html>`_ will be generated in
the following form:

.. sourcecode:: erlang

    error:{ruby, ExceptionClass, ExceptionArgument, ReversedStackTrace}

For example:

.. sourcecode:: erl

    4> try ruby:call(R, unknown, unknown, [])
    4> catch error:{ruby, Class, Argument, StackTrace} -> error
    4> end.
    error
    5> Class.
    'LoadError'
    6> Argument.
    <<"no such file to load -- unknown">>
    7> StackTrace.
    [<<"-e:1">>,<<"-e:1:in `require'">>,
     <<"/../erlport/priv/ruby1.8/erlport/cli.rb:94">>,
     <<"/../erlport/priv/ruby1.8/erlport/cli.rb:41:in `main'">>,
     <<"/../erlport/priv/ruby1.8/erlport/erlang.rb:135:in `start'">>,
     <<"/../erlport/priv/ruby1.8/erlport/erlang.rb:191:in `_receive'">>,
     <<"/../erlport/priv/ruby1.8/erlport/erlang.rb:231:in `call_with_e"...>>,
     <<"/../erlport/priv/ruby1.8/erlport/erlang.rb:192:in `_receiv"...>>,
     <<"/../erlport/priv/ruby1.8/erlport/erlang.rb:215:in `inc"...>>,
     <<"/../erlport/priv/ruby1.8/erlport/erlang.rb:215:in "...>>]

And of course don't forget to stop the instance at the end:

.. sourcecode:: erl

    8> ruby:stop(R).
    ok

If you want to call a function from your own Ruby file in most cases you need
to set the `Ruby lib`_. You can do it with `ruby:start/1`_ function or
*RUBYLIB* `environment variable`_. The `ruby:start/1`_ also can be used to
change the default Ruby interpreter. For example let's create a simple Ruby
file ``/path/to/my/modules/version.rb``:

.. sourcecode:: ruby

    def version
        "#{RUBY_VERSION}-p#{RUBY_PATCHLEVEL}"
    end

Now we can set path to this module in `ruby:start/1`_ like this:

.. sourcecode:: erl

    1> {ok, R} = ruby:start([{ruby_lib, "/path/to/my/modules"},
    1>                       {ruby, "ruby1.9.3"}]).
    {ok,<0.34.0>}
    2> ruby:call(R, version, version, []).
    <<"1.9.3-p0">>
    3> ruby:stop(R).
    ok

Call Erlang functions from Ruby
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: Ruby call API

    Check `Ruby API`_ and `data types mapping`_ sections for more details
    about `ErlPort::Erlang::call()`_ function and supported data types.

ErlPort uses Ruby ``erlport/erlang.rb`` file with ``ErlPort::Erlang`` module as
an interface to Erlang. Namely `ErlPort::Erlang::call()`_ function allows to
call Erlang functions from Ruby. The function accepts Module and Function
arguments as ``Symbol()`` (and `ErlPort::ErlTerm::EmptySymbol()`_ for Ruby
1.8.*) object and Arguments as an Array(). Currently each Erlang function will
be called in a new Erlang process. Let's create the following Ruby module in
``pids.rb`` file in the current directory which will be added to Ruby lib path
automatically by Ruby:

.. sourcecode:: ruby

    include ErlPort::Erlang

    def pids
        pid1 = call(:erlang, :self, [])
        pid2 = call(:erlang, :self, [])
        [pid1, pid2]
    end

Now we can call this function from Erlang:

.. sourcecode:: erl

    1> {ok, R} = ruby:start().
    {ok,<0.34.0>}
    2> ruby:call(R, pids, pids, []).
    [<0.36.0>,<0.37.0>]
    3> ruby:stop(R).
    ok

To simplify the demonstration the next example will use the call chaining so
Ruby to Erlang calls will be initiated from Erlang shell. The following example
also demonstrate the communication between two Ruby instances:

.. sourcecode:: erl


    1> {ok, R1} = ruby:start().
    {ok,<0.34.0>}
    2> {ok, R2} = ruby:start().
    {ok,<0.36.0>}
    3> ruby:call(R1, '', 'Process::pid', []).
    5196
    4> ruby:call(R2, '', 'Process::pid', []).
    5198
    5> ruby:call(R1, 'erlport/erlang', call,
    5>           [ruby, call, [R2, '', 'Process::pid', []]]).
    5198
    6> ruby:stop(R1).
    ok
    7> ruby:stop(R2).
    ok

So the command #5 actually calls `ErlPort::Erlang::call()`_ function for
instance ``R1``, which calls Erlang function `ruby:call/4`_, which in order
calls Ruby function ``Process::pid()`` for instance ``R2``.

Send messages from Erlang to Ruby
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: Erlang cast API

    Check `Erlang API`_, `Ruby API`_ and `data types mapping`_ sections for
    more details about `ruby:cast/2`_ and
    `ErlPort::Erlang::set_message_handler()`_ functions and supported data
    types.

To send a message from Erlang to Ruby first a message handler function on Ruby
side should be set. The message handler function can be set with
`ErlPort::Erlang::set_message_handler()`_ function. The default message handler
just ignore all the incoming messages. And if you don't need to handle incoming
message anymore the default handler can be set again with
`ErlPort::Erlang::set_default_message_handler()`_ function.

*Be careful when you write a message handling function because the function can
also get some unexpected messages which probably should be ignored and in case
of any error in the message handler the whole instance will be shut down.*

To demonstrate message sending from Erlang to Ruby we will first create the
following module in the current directory in a file ``handler.rb``:

.. sourcecode:: ruby

    include ErlPort::Erlang

    def register_handler dest
        set_message_handler {|message|
            cast dest, message
        }
        :ok
    end

This message handler just send all messages to the selected Erlang process.

To send a message to Ruby `ruby:cast/2`_ function can be used and also all
unknown to ErlPort messages will be redirected to the message handler.

.. sourcecode:: erl

    1> {ok, R} = ruby:start().
    {ok,<0.34.0>}
    2> ruby:call(R, handler, register_handler, [self()]).
    ok
    3> ruby:cast(R, test_message).
    ok
    4> flush().
    Shell got test_message
    ok
    5> R ! test_message2.
    test_message2
    6> flush().
    Shell got test_message2
    ok
    7> ruby:stop(R).
    ok

Send messages from Ruby to Erlang
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: Ruby cast API

    Check `Ruby API`_ and `data types mapping`_ sections for more details about
    `ErlPort::Erlang::cast()`_ function and supported data types.

It's very easy to send a message from Ruby to Erlang - you just need to know
the ``pid()`` or registered name of the destination process. The function
`ErlPort::Erlang::cast()`_ accepts two arguments - the id of the destination
process and a message which can be any supported data type according to `Data
types mapping`_. And of course you can send messages to any other ErlPort
process.

The following is a demonstration of message sending from Ruby:

.. sourcecode:: erl

    1> {ok, R} = ruby:start().
    {ok,<0.34.0>}
    2> ruby:call(R, 'erlport/erlang', cast, [self(), test_message]).
    undefined
    3> flush().
    Shell got test_message
    ok
    4> register(test_process, self()).
    true
    5> ruby:call(R, 'erlport/erlang', cast, [test_process, test_message2]).
    undefined
    6> flush().
    Shell got test_message2
    ok
    7> ruby:stop(R).
    ok

Custom data types
~~~~~~~~~~~~~~~~~

.. sidebar:: Ruby data types API

    Check `Ruby API`_ and `data types mapping`_ sections for more details about
    `ErlPort::Erlang::set_encoder()`_ and `ErlPort::Erlang::set_decoder()`_
    functions and supported data types.

ErlPort only supports a minimal `set of data types`_ to make sure the types are
orthogonal - can be created and meaningful in any language supported by
ErlPort. In addition ErlPort also supports language specific opaque data type
containers so for example Ruby instances can exchange any `serializable`_ data
type. But sometimes it's better to use *rich* inter-language data types in
which case custom data types can be used.

There are two functions to support custom data types:

- `ErlPort::Erlang::set_encoder()`_ which sets the Ruby to Erlang data type
  converter, and
- `ErlPort::Erlang::set_decoder()`_ which sets the converter for the opposite
  direction - Erlang to Ruby

Both of the functions can be reset to the default, which just pass the value
unmodified, with `ErlPort::Erlang::set_default_encoder()`_ and
`ErlPort::Erlang::set_default_decoder()`_ functions correspondingly.

To give you a feeling how it works the following file in the current directory
with name ``date_type.rb`` will add the partial support to ErlPort for
`Time()`_ objects:

.. sourcecode:: ruby

    include ErlPort::ErlTerm
    include ErlPort::Erlang

    def setup_date_type
        set_encoder {|v| date_encoder v}
        set_decoder {|v| date_decoder v}
        :ok
    end

    def date_encoder value
        if value.is_a? Time
            value = Tuple.new([:date,
                Tuple.new([value.year, value.month, value.day])])
        end
        value
    end

    def date_decoder value
        if value.is_a? Tuple and value.length == 2 and value[0] == :date
            year, month, day = value[1]
            value = Time.utc(year, month, day)
        end
        value
    end

    def add date, sec
        date + sec
    end

The ``date_type`` module can be used in Erlang shell like this:

.. sourcecode:: erl

    1> {ok, R} = ruby:start().
    {ok,<0.34.0>}
    2> ruby:call(R, date_type, setup_date_type, []).
    ok
    3> Date = ruby:call(R, '', 'Time::utc', [2012, 12, 31]).
    {date,{2012,12,31}}
    4> ruby:call(R, date_type, add, [Date, 60 * 60 * 24]).
    {date,{2013,1,1}}
    5> ruby:stop(R).
    ok

Standard output redirection
~~~~~~~~~~~~~~~~~~~~~~~~~~~

As a convenient feature ErlPort also supports redirection of Ruby`s `STDOUT`_
to Erlang which can be used for example for debugging. For example:

.. sourcecode:: erl

    1> {ok, R} = ruby:start().
    {ok,<0.34.0>}
    2> ruby:call(R, '', puts, [<<"Hello, World!">>]).
    Hello, World!
    undefined
    3> ruby:stop(R).
    ok

Reference manual
----------------

Here you can find complete description of `data types mapping`_, `Erlang
functions`_, `Ruby functions`_ and `environment variables`_ supported by
ErlPort.

.. _set of data types:

Data types mapping
~~~~~~~~~~~~~~~~~~

The following table defines mapping of Erlang data types to Ruby data types:

+--------------------------------------+--------------------------------------+
| Erlang data type                     | Ruby data type                       |
+======================================+======================================+
| integer()                            | Integer()                            |
+--------------------------------------+--------------------------------------+
| float()                              | Float()                              |
+--------------------------------------+--------------------------------------+
| atom()                               | Symbol() and                         |
|                                      | `ErlPort::ErlTerm::EmptySymbol()`_   |
|                                      | in Ruby 1.8.*                        |
+--------------------------------------+--------------------------------------+
| true                                 | true                                 |
+--------------------------------------+--------------------------------------+
| false                                | false                                |
+--------------------------------------+--------------------------------------+
| undefined                            | nil                                  |
+--------------------------------------+--------------------------------------+
| binary()                             | String()                             |
+--------------------------------------+--------------------------------------+
| tuple()                              | `ErlPort::ErlTerm::Tuple()`_         |
+--------------------------------------+--------------------------------------+
| list()                               | Array()                              |
+--------------------------------------+--------------------------------------+
| improper_list()                      | `ErlPort::ErlTerm::ImproperList()`_  |
+--------------------------------------+--------------------------------------+
| *Opaque Ruby data type container*    | *Ruby data type*                     |
+--------------------------------------+--------------------------------------+
| *Opaque data type container*         | *Opaque data type container*         |
+--------------------------------------+--------------------------------------+

And here is the table of Ruby to Erlang data types mapping. The types mapping
between Erlang and Ruby are practically orthogonal:

+--------------------------------------+--------------------------------------+
| Ruby data type                       | Erlang data type                     |
+======================================+======================================+
| Integer()                            | integer()                            |
+--------------------------------------+--------------------------------------+
| Float()                              | float()                              |
+--------------------------------------+--------------------------------------+
| Symbol() and                         | atom()                               |
| `ErlPort::ErlTerm::EmptySymbol()`_   |                                      |
| in Ruby 1.8.*                        |                                      |
+--------------------------------------+--------------------------------------+
| true                                 | true                                 |
+--------------------------------------+--------------------------------------+
| talse                                | false                                |
+--------------------------------------+--------------------------------------+
| nil                                  | undefined                            |
+--------------------------------------+--------------------------------------+
| String()                             | binary()                             |
+--------------------------------------+--------------------------------------+
| `ErlPort::ErlTerm::Tuple()`_         | tuple()                              |
+--------------------------------------+--------------------------------------+
| Array()                              | list()                               |
+--------------------------------------+--------------------------------------+
| `ErlPort::ErlTerm::ImproperList()`_  | improper_list()                      |
+--------------------------------------+--------------------------------------+
| *Other Ruby data type*               | *Opaque Ruby data type container*    |
+--------------------------------------+--------------------------------------+
| *Opaque data type container*         | *Opaque data type container*         |
+--------------------------------------+--------------------------------------+

The following classes can be found in ``erlport/erlterms.rb`` file.

.. _ErlPort::ErlTerm::EmptySymbol():

ErlPort::ErlTerm::EmptySymbol()
    Class to represent empty Erlang atoms in Ruby 1.8.*. Empty symbols support
    was added to Ruby in 1.9.1.

.. _ErlPort::ErlTerm::Tuple():

ErlPort::ErlTerm::Tuple(array)
    Class to represent Erlang tuples in Ruby. Basically just a subclass of
    Array().

.. _ErlPort::ErlTerm::ImproperList():

ErlPort::ErlTerm::ImproperList(array, tail)
    Class to represent Erlang improper lists in Ruby. The ``tail`` argument
    can't be an array. *Note that this class exists mostly to convert improper
    lists received from Erlang side and probably there are no reasons to create
    instances of this class in Ruby.*

.. _Erlang functions:

Erlang API
~~~~~~~~~~

.. _ruby:start/0:

ruby:start() -> {ok, Pid} | {error, Reason}
    Start Ruby instance with the default options

.. _ruby:start/1:
.. _ruby_lib:
.. _env:

ruby:start(Options) -> {ok, Pid} | {error, Reason}
    Start Ruby instance with options. The ``Options`` argument should be
    a list with the following options.

    General options:

    {buffer_size, Size::pos_integer()}
        Size in bytes of the ErlPort receive buffer on Ruby side. The default
        is 65536 bytes.
    {call_timeout, Timeout::pos_integer() | infinity}
        Default timeout in milliseconds for function calls. Per call timeouts
        can be set with `ruby:call/5`_ function.
    {cd, Path::string()}
        Change current directory to ``Path`` before starting.
    {compressed, 0..9}
        Set terms compression level. `0` means no compression and `9` will take
        the most time and *may (or may not)* produce a smaller result. Can be
        used as an optimisation if you know that your data can be easily
        compressed.
    {env, [{Name::string(), Value::string() | false}]}
        Set environment for Ruby instance. The ``Name`` variable is the name
        of environment variable to set and ``Value`` can be a string value of
        the environment variable or ``false`` if the variable should be
        removed.
    nouse_stdio
        Not use `STDIN/STDOUT`_ for communication. *Not supported on Windows.*
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
        Use `STDIN/STDOUT`_ for communication. The default.

    Ruby related options:

    {ruby, Ruby::string()}
        Path to the Ruby interpreter executable
    {ruby_lib, Path::string() | [Path::string()]}
        The Ruby programs search path. The ``Path`` variable can be a string in
        `RUBYLIB`_ format or a list of paths. The priorities of different ways
        to set the modules search path is as follows:

        #. `ruby_lib`_ option
        #. *RUBYLIB* environment variable set through the `env`_ option
        #. *RUBYLIB* environment variable

.. _ruby:start/2:

ruby:start(Name, Options) -> {ok, Pid} | {error, Reason}
    Start named Ruby instance. The instance will be registered with ``Name``
    name. The ``Options`` variable is the same as for `ruby:start/1`_.

.. _ruby:start_link/0:

ruby:start_link() -> {ok, Pid} | {error, Reason}
    The same as `ruby:start/0`_ except the link to the current process is
    also created.

.. _ruby:start_link/1:

ruby:start_link(Options) -> {ok, Pid} | {error, Reason}
    The same as `ruby:start/1`_ except the link to the current process is
    also created.

.. _ruby:start_link/2:

ruby:start_link(Name, Options) -> {ok, Pid} | {error, Reason}
    The same as `ruby:start/2`_ except the link to the current process is
    also created.

.. _ruby:stop/1:

ruby:stop(Instance) -> ok
    Stop Ruby instance

.. _ruby:call/4:

ruby:call(Instance, File, Function, Arguments) -> Result
    Call Ruby function. The ``Instance`` variable can be a ``pid()`` which
    returned by one of the ``ruby:start`` functions or an instance name
    (atom()) if the instance was registered with a name. The ``File`` and
    ``Function`` variables should be atoms and ``Arguments`` is a list.

    In case of any error on Ruby side during the function call an exception
    of class `error <http://www.erlang.org/doc/reference_manual/errors.html>`_
    will be generated in the following form:

    .. sourcecode:: erlang

        error:{ruby, ExceptionClass, ExceptionArgument, ReversedStackTrace}

.. _ruby:call/5:

ruby:call(Instance, File, Function, Arguments, Options) -> Result
    The same as `ruby:call/4`_ except the following options can be added:

    {timeout, Timeout::pos_integer() | infinity}
        Call timeout in milliseconds.

.. _ruby:cast/2:

ruby:cast(Instance, Message) -> ok
    Send a message to the Ruby instance.

.. _Ruby functions:

Ruby API
~~~~~~~~

All the following functions can be found in ``erlport/erlang.rb`` file.

.. _ErlPort::Erlang::call():

ErlPort::Erlang::call(module, function, arguments) -> result
    Call Erlang function as ``module:function(arguments)``. The ``function``
    and ``module`` variables should be of type ``Symbol`` and ``arguments``
    should be an ``Array``.

.. _ErlPort::Erlang::cast():

ErlPort::Erlang::cast(pid, message)
    Send a message to Erlang. The ``pid`` and ``message`` variables should be
    the same types as supported by `Erlang ! (send) expression
    <http://www.erlang.org/doc/reference_manual/expressions.html#id77156>`_.
    Erlang ``pid()`` variables however can't be created in Ruby but can be
    passed as parameters from Erlang.

.. _ErlPort::Erlang::self():

ErlPort::Erlang::self() -> pid
    Get the Erlang pid of the Ruby instance

.. _ErlPort::Erlang::set_encoder():

ErlPort::Erlang::set_encoder(&encoder)
    Set encoder for custom data types. Encoder is a code block with a single
    ``value`` argument which is can be any Ruby data type and should return an
    Erlang representation of this type using supported `Data types mapping`_.

.. _ErlPort::Erlang::set_decoder():

ErlPort::Erlang::set_decoder(&decoder)
    Set decoder for custom data types. Decoder is a code block with a single
    ``value`` argument which is one of the supported Erlang data types
    according to `Data types mapping`_. The function should decode and return
    Erlang representation of the *rich* Ruby data type.

.. _ErlPort::Erlang::set_message_handler():

ErlPort::Erlang::set_message_handler(&handler)
    Set message handler. Message handler is a code block with a single
    ``message`` argument which receive all the incoming messages.

.. _ErlPort::Erlang::set_default_encoder():

ErlPort::Erlang::set_default_encoder()
    Reset custom data types encoder to the default which is just pass the term
    through without any modifications

.. _ErlPort::Erlang::set_default_decoder():

ErlPort::Erlang::set_default_decoder()
    Reset custom data types decoder to the default which is just pass the term
    through without any modifications

.. _ErlPort::Erlang::set_default_message_handler():

ErlPort::Erlang::set_default_message_handler()
    Reset message handler to the default which is just ignore all the incoming
    messages

.. _environment variables:
.. _environment variable:

Environment variables
~~~~~~~~~~~~~~~~~~~~~

The following environment variables can change the default behavior of
ErlPort:

ERLPORT_RUBY
    Path to Ruby interpreter executable which will be used by default.

RUBYLIB
    The default search patch for Ruby programs. The same as `RUBYLIB`_
    environment variable supported by Ruby. The priorities of different ways to
    set the programs search path is as follows:

    #. `ruby_lib`_ option
    #. *RUBYLIB* environment variable set through the `env`_ option
    #. *RUBYLIB* environment variable



.. _RUBYLIB: http://www.ruby-doc.org/docs/ProgrammingRuby/html/rubyworld.html
.. _Ruby lib: `RUBYLIB`_
.. _STDIN/STDOUT: http://en.wikipedia.org/wiki/Standard_streams
.. _STDOUT: `STDIN/STDOUT`_
.. _serializable: http://www.ruby-doc.org/core-2.0/Marshal.html
.. _Time(): http://www.ruby-doc.org/core-2.0/Time.html
