ErlPort - Downloads
===================

.. meta::
   :keywords: erlport erlang python ruby downloads
   :description: Downloads for ErlPort library

ErlPort downloads
+++++++++++++++++

.. sidebar:: ErlPort prerequisites

    `Erlang <http://erlang.org>`__
        R13 or higher
    `Python <http://python.org>`__
        2.5 or higher (including Python 3)
    `Ruby <http://ruby-lang.org>`__
        1.8.6 or higher (including Ruby 2)

.. contents::

Installation
------------

The easiest method to install ErlPort is by using one of the `binary packages
</downloads/#binary-packages>`__. Select the package for your version of Erlang
and unpack it to an Erlang library directory. The default Erlang library
directory can be found with `code:lib_dir/0
<http://www.erlang.org/doc/man/code.html#lib_dir-0>`_ function. Please check
the *"Code Path"* section of the `code module documentation
<http://www.erlang.org/doc/man/code.html>`_ for more details about the library
directory.

ErlPort library also can be run or installed from the `source packages
</downloads/#source-packages>`__ or https://github.com/hdima/erlport
repository. To install ErlPort from a source package just unpack the package to
a directory of your choice. Or you can use `Git <http://git-scm.com>`__ to
clone the ErlPort repository with the following command:

.. sourcecode:: sh

    $ git clone https://github.com/hdima/erlport.git

After you unpacking the sources either by using one of the source packages or
cloning the repository you can use `make
<http://en.wikipedia.org/wiki/Make_%28software%29>`__ command to build them:

.. sourcecode:: sh

    $ make

To build the binary packages just issue the following command which will
create the *packages* directory with new packages:

.. sourcecode:: sh

    $ make release

If you just want to run ErlPort from the source directory you can start `Erlang
shell <http://www.erlang.org/doc/man/shell.html>`__ with ErlPort with the
following command:

.. sourcecode:: sh

    $ erl -env ERL_LIBS ../erlport

Please check `Documentation </docs>`_ page for features of ErlPort, examples
and more details of how to use the library.

ErlPort 1.0.0alpha
------------------

Make sure you read `Installation instruction <#installation>`__ before
downloading any of the packages below.

.. class:: warning

*WARNING: It's still an alpha version so expect bugs and backward incompatible
changes in the future*

Binary packages
~~~~~~~~~~~~~~~

The following are the binary packages compiled for each supported version of
Erlang and packaged into *tar.gz* and *zip* archives:

+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *tar.gz* archive for Erlang R16 | `<erlport-R16-0.9.8.tar.gz>`__ (76K) |
+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *zip* archive for Erlang R16    | `<erlport-R16-0.9.8.zip>`__ (126K)   |
+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *tar.gz* archive for Erlang R15 | `<erlport-R15-0.9.8.tar.gz>`__ (76K) |
+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *zip* archive for Erlang R15    | `<erlport-R15-0.9.8.zip>`__ (126K)   |
+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *tar.gz* archive for Erlang R14 | `<erlport-R14-0.9.8.tar.gz>`__ (75K) |
+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *zip* archive for Erlang R14    | `<erlport-R14-0.9.8.zip>`__ (125K)   |
+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *tar.gz* archive for Erlang R13 | `<erlport-R13-0.9.8.tar.gz>`__ (75K) |
+----------------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *zip* archive for Erlang R13    | `<erlport-R13-0.9.8.zip>`__ (125K)   |
+----------------------------------------------------+--------------------------------------+

Source packages
~~~~~~~~~~~~~~~

The ErlPort source archives in *tar.gz* and *zip* formats:

+---------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *tar.gz* sources archive | `<erlport-src-0.9.8.tar.gz>`__ (70K) |
+---------------------------------------------+--------------------------------------+
| ErlPort 1.0.0alpha *zip* sources archive    | `<erlport-src-0.9.8.zip>`__ (153K)   |
+---------------------------------------------+--------------------------------------+
