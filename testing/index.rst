Remote Testing Tools
====================

The ``openmdao.devtools`` package contains a number of console scripts that
allow openmdao to be tested and built on remote hosts. This section describes
how to setup and use the scripts.

General Setup
-------------

Information about remote hosts is contained in a config file.  An example
of such a file is ``config/testhosts.cfg`` in the 
OpenMDAO-Framework repository.  This file should be copied to
``~/.openmdao/testhosts.cfg`` and modified to contain the hosts or EC2 images
you intend to test on.  The scripts look for this file in ``~/.openmdao``
by default.  You can specify a different config file on the command line using
the ``-c`` argument.

Aside from the [DEFAULT] section, the file has one section per 
host or EC2 image.  The section name is used as a short alias for that host 
and is used with the ``--host=<section_name>`` arg in the testing scripts.


EC2 Specific Setup
------------------

To run the scripts on EC2 images or non-running instances, you must create
a ``~/.boto``  config file with the appropriate id and secret key.  You may
also specify other information in the ``.boto file``, e.g., debug level.  An
example of a ``.boto`` file is shown below.


::

    [Credentials]
    aws_access_key_id = <your id here>
    aws_secret_access_key = <your secret key here>
    
    [Boto]
    debug = 0
    num_retries = 5
    
    #proxy = myproxy.com
    #proxy_port = 8080
    #proxy_user = <your proxy userid>
    #proxy_pass = <your proxy password>


SSH keys
~~~~~~~~

You'll need an identity file to execute operations like starting and
stopping instances on EC2 using the *boto* package. For OpenMDAO
we use an identity file called ``lovejoy.pem`` for all of our EC2 images
and instances. The identity file should be placed in the ``~/.ssh`` directory.

To actually connect to a given host via SSH, you'll need to take
your personal public key for the host you're connecting from and put it
in the ``authorized_keys`` file on the destination host.  This is true whether
the host is an EC2 host or not.


Scripts
-------

The following section describes each script in detail. All scripts accept the
``-h`` and the ``--help`` command-line options which will display all of their
allowed arguments.


test_branch
~~~~~~~~~~~

The ``test_branch`` script is used to test a branch running ``openmdao_test`` 
on a group of remote hosts. Running it with a ``-h`` option will display the following:

::

    Usage: test_branch [OPTIONS] -- [options to openmdao_test]

    Options:
       -h, --help     show this help message and exit
       -c CONFIG, --config=CONFIG
                      Path of config file where info for hosts is located
       --host=HOST    Select host from config file to run on. To run on
                      multiple hosts, use multiple --host args.
       --all          If True, run on all hosts in config file.
       -o OUTDIR, --outdir=OUTDIR
                      Output directory for results (defaults to ./host_results)
       -k, --keep     If there are test/build failures, don't delete the
                      temporary build directory. If testing on EC2, stop 
                      the instance instead of terminating it. 
       -f FNAME, --file=FNAME
                      Pathname of a tarfile or URL of a Git repo. 
                      Defaults to the current repo.
       -b BRANCH, --branch=BRANCH
                      If file is a Git repo, supply branch name here


The tests run concurrently and write their outputs to 
``<outdir>/<host_config_name>/run.out`` where ``outdir`` defaults to ``host_results``,
and ``host_config_name`` is the section name for that host in the config file.

The ``--host`` arg can be used multiple times to specify more than one host.

The script can test the current (committed) branch of a Git repository, 
a tarred repository, or a specific branch of a specified local or remote Git 
repository.  If a Git repository is specified rather than a tar file, then
the branch must also be specified. If no ``-f`` is supplied, the current
branch of the current repository is used.

If a ``--`` arg is supplied, any args after that are passed to openmdao_test
on the remote host.  Adding a ``-x`` arg after the ``--`` arg, for example, 
would cause the test to end as soon as any test on the remote host failed.
Adding the name of a specific module to test can also be a big time saver
when debugging a specific test failure.


test_release
~~~~~~~~~~~~

The ``test_release`` script is used to test a release by running ``openmdao_test``
on a group of remote hosts.  It can also be used to test an existing 
production release on a specific host. Running it with a ``-h`` option 
will display the following:


::

    Usage: test_release [OPTIONS] -- [options to openmdao_test]

    Options:
      -h, --help        show this help message and exit
      -c CONFIG, --config=CONFIG
                        Path of config file where info for hosts is located
      --host=HOST       Select host from config file to run on. To run on
                        multiple hosts, use multiple --host args
      --all             If True, run on all hosts in config file.
      -o OUTDIR, --outdir=OUTDIR
                        Output directory for results (defaults to
                        ./host_results)
      -k, --keep        Don't delete the temporary build directory. If testing
                        on EC2 stop the instance instead of terminating it.
      -f FNAME, --file=FNAME
                        URL or pathname of a go-openmdao.py file or pathname
                        of a release dir

The ``-f`` argument is used to specify either the ``go-openmdao.py`` file that 
builds the release environment or the path to a directory that was built 
using the ``make_release`` script.

