.. class:: news

*2013-06-DD* Released ErlPort 1.0.0alpha
  After almost 2 years of development new `1.0.0alpha
  </downloads/#erlport-1-0-0alpha>`__ version of ErlPort library was released.

  New version of ErlPort helps develop applications which need to connect
  Erlang to Python or Ruby or need to use Erlang as a middleware for Python or
  Ruby.

  The following is an example session with ErlPort and Python:

  .. sourcecode:: erl

    1> {ok, P} = python:start().
    {ok,<0.34.0>}
    2> python:call(P, sys, 'version.__str__', []).
    <<"2.7.3 (default, Aug  1 2012, 05:14:39) \n[GCC 4.6.3]">>
    3> python:call(P, operator, add, [2, 2]).
    4
    4> python:stop(P).
    ok

  **TODO: Warning about alpha version!**

  Please check `documentation </docs>`__ page for more details about ErlPort
  and `downloads </downloads>`__ page for more download options.
