|logo| Pylama
#############

.. _description:

Code audit tool for Python and JavaScript. Pylama wraps these tools:

* PEP8_ © 2012-2013, Florent Xicluna;
* PEP257_  © 2012, GreenSteam, <http://greensteam.dk/>
* PyFlakes_ © 2005-2013, Kevin Watters;
* Mccabe_ © Ned Batchelder;
* Pylint_ © 2013, Logilab (should be installed 'pylama_pylint' module);
* gjslint_ © The Closure Linter Authors (should be installed 'pylama_gjslint' module);

 |  `Pylint doesnt supported in python3.`

.. _badges:

.. image:: https://secure.travis-ci.org/klen/pylama.png?branch=develop
    :target: http://travis-ci.org/klen/pylama
    :alt: Build Status

.. image:: https://coveralls.io/repos/klen/pylama/badge.png
    :target: https://coveralls.io/r/klen/pylama
    :alt: Coverals

.. image:: https://pypip.in/v/pylama/badge.png
    :target: https://crate.io/packages/pylama
    :alt: Version

.. image:: https://pypip.in/d/pylama/badge.png
    :target: https://crate.io/packages/pylama
    :alt: Downloads

.. image:: https://dl.dropboxusercontent.com/u/487440/reformal/donate.png
    :target: https://www.gittip.com/klen/
    :alt: Donate


.. _documentation:

Docs are available at https://pylama.readthedocs.org/. Pull requests with documentation enhancements and/or fixes are awesome and most welcome.


.. _contents:

.. contents::

.. _requirements:

Requirements:
=============

- Python (2.6, 2.7, 3.2, 3.3)
- To use JavaScript checker (``gjslint``) you need to install ``python-gflags`` with ``pip install python-gflags``.
- If your tests are failing on Win platform you are missing: ``curses`` - http://www.lfd.uci.edu/~gohlke/pythonlibs/
  (The curses library supplies a terminal-independent screen-painting and keyboard-handling facility for text-based terminals)


.. _installation:

Instalation:
============
**Pylama** could be installed using pip: ::
::

    $ pip install pylama


.. _quickstart:

Quickstart
==========

**Pylama** is easy to use and realy fun for checking code quality.
Just run `pylama` and get common output from all pylama plugins (PEP8_, PyFlakes_ and etc)

Recursive check the current directory. ::

    $ pylama

Recursive check a path. ::

    $ pylama <path_to_directory_or_file>

Ignore errors ::

    $ pylama -i W,E501

.. note:: You could choose a group erros `D`,`E1` and etc or special errors `C0312`

Choose code checkers ::

    $ pylama -l "pep8,mccabe"

Choose code chekers for JavaScript::

    $ pylama --linters=gjslint --ignore=E:0010 <path_to_directory_or_file>

.. _options:

Set Pylama (checkers) options
=============================

Command line options
--------------------

::

    $ pylama --help

    usage: pylama [-h] [--verbose] [--version] [--format {pep8,pylint}]
                [--select SELECT] [--linters LINTERS] [--ignore IGNORE]
                [--skip SKIP] [--report REPORT] [--hook] [--async]
                [--options OPTIONS] [--force]
                [path]

    Code audit tool for python.

    positional arguments:
    path                  Path on file or directory for code check.

    optional arguments:
    -h, --help            show this help message and exit
    --verbose, -v         Verbose mode.
    --version             show program's version number and exit
    --format {pep8,pylint}, -f {pep8,pylint}
                            Choose errors format (pep8, pylint).
    --select SELECT, -s SELECT
                            Select errors and warnings. (comma-separated list)
    --linters LINTERS, -l LINTERS
                            Select linters. (comma-separated).
    --ignore IGNORE, -i IGNORE
                            Ignore errors and warnings. (comma-separated list)
    --skip SKIP           Skip files by masks (comma-separated, Ex.
                            */messages.py)
    --report REPORT, -r REPORT
                            Send report to file [REPORT]
    --hook                Install Git (Mercurial) hook.
    --async               Enable async mode. Usefull for checking a lot of
                            files. Dont supported with pylint.
    --options OPTIONS, -o OPTIONS
                            Select configuration file. By default is
                            '<CURDIR>/pylama.ini'
    --force, -F           Force code checking (if linter doesnt allow)

.. _modeline:

File modelines
--------------

You can set options for **Pylama** inside a source files. Use
pylama *modeline* for this.

Format: ::

    # pylama:{name1}={value1}:{name2}={value2}:...


::

     .. Somethere in code
     # pylama:ignore=W:select=W301


Disable code checking for current file: ::

     .. Somethere in code
     # pylama:skip=1

The options have a must higher priority.

.. _skiplines:

Skip lines (noqa)
-----------------

Just add `# noqa` in end of line for ignore.

::

    def urgent_fuction():
        unused_var = 'No errors here' # noqa


.. _config:

Configuration files
-------------------

When starting **Pylama** try loading configuration file.

The programm searches for the first matching ini-style configuration file in
the directories of command line argument. Pylama looks for the configuration
in this order: ::

    pylama.ini
    setup.cfg
    tox.ini
    pytest.ini

You could set configuration file manually by "-o" option.

Pylama search sections with name starts `pylama`.

Section `pylama` contains a global options, like `linters` and `skip`.

::

    [pylama]
    format = pylint
    skip = */.tox/*,*/.env/*
    linters = pylint,mccabe
    ignore = F0401,C0111,E731

Set Code-checkers' options
--------------------------

You could set options for special code checker with pylama configurations.

::

    [pylama:pyflakes]
    builtins = _

    [pylama:pep8]
    max_line_length = 100

    [pylama:pylint]
    max_line_length = 100
    disable = R

See code checkers documentation for more info.


Set options for file (group of files)
-------------------------------------

You could set options for special file (group of files)
with sections:

The options have a higher priority than in the `pylama` section.

::

    [pylama:*/pylama/main.py]
    ignore = C901,R0914,W0212
    select = R

    [pylama:*/tests.py]
    ignore = C0110

    [pylama:*/setup.py]
    skip = 1


Pytest integration
==================

Pylama have Pytest_ support. The package automatically register self as pytest
plugin when during installation. Also pylama suports `pytest_cache` plugin.

Check files with pylama ::

    pytest --pylama ...

Recomended way to settings pyalam options when using pytest — configuration
files (see below).


Writing a linter
================

You can write a custom extension for Pylama.
Custom linter should be a python module. Name should be like 'pylama_<name>'.

In 'setup.py' should be defined 'pylama.linter' entry point. ::

    setup(
        # ...
        entry_points={
            'pylama.linter': ['lintername = pylama_lintername.main:Linter'],
        }
        # ...
    ) 

'Linter' should be instance of 'pylama.lint.Linter' class.
Must implemented two methods:

'allow' take a path and returned true if linter could check this file for errors.
'run' take a path and meta keywords params and return list of errors.

Example:
--------

Just virtual 'WOW' checker.

setup.py: ::

    setup(
        name='pylama_wow',
        install_requires=[ 'setuptools' ],
        entry_points={
            'pylama.linter': ['wow = pylama_wow.main:Linter'],
        }
        # ...
    ) 

pylama_wow.py: ::

    from pylama.lint import Linter as BaseLinter

    class Linter(BaseLinter):

        def allow(self, path):
            return 'wow' in path

        def run(self, path, **meta):
            with open(path) as f:
                if 'wow' in f.read():
                    return [{
                        lnum: 0,
                        col: 0,
                        text: 'Wow has been finded.',
                        type: 'WOW'
                    }]


.. _bagtracker:

Bug tracker
-----------

If you have any suggestions, bug reports or annoyances please report them to the issue tracker at https://github.com/klen/pylama/issues


.. _contributing:

Contributing
------------

Development of adrest happens at github: https://github.com/klen/pylama


.. _contributors:

Contributors
^^^^^^^^^^^^

See AUTHORS_.


.. _license:

License
-------

Licensed under a `BSD license`_.


.. _links:

.. _AUTHORS: https://github.com/klen/pylama/blob/develop/AUTHORS 
.. _BSD license: http://www.linfo.org/bsdlicense.html
.. _Mccabe: http://nedbatchelder.com/blog/200803/python_code_complexity_microtool.html
.. _PEP257: https://github.com/GreenSteam/pep257
.. _PEP8: https://github.com/jcrocholl/pep8
.. _PyFlakes: https://github.com/kevinw/pyflakes 
.. _Pylint: http://pylint.org
.. _Pytest: http://pytest.org
.. _gjslint: https://developers.google.com/closure/utilities
.. _klen: http://klen.github.io/
.. |logo| image:: https://raw.github.com/klen/pylama/develop/docs/_static/logo.png
                  :width: 100
