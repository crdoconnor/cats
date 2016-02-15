Cookiecutter-Django Set up
==========================

Set up::

    $ curl https://raw.githubusercontent.com/mitsuhiko/pipsi/master/get-pipsi.py | python

    # It looks like ~/.local/bin/ is not on your PATH?

    $ echo PATH=~/.local/bin/:$PATH >> ~/.bashrc

    $ pipsi install cookiecutter

    $ cookiecutter https://github.com/pydanny/cookiecutter-django.git

    $ cd tests/



Run tests
---------

Run tests::

    $ pipsi install hitch

    $ hitch init

    $ hitch test register-and-log-in.test