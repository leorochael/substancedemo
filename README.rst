substancedemo
=============

This packages is a demonstration of deploying SubstanceD to Heroku. It's not
meant as the ultimate standard for doing so, only as a proof of concept and
starting point.

It assumes you have already installed the
`heroku toolbelt <https://toolbelt.heroku.com/>`_ for your platform
and that you're familiar with
`virtualenv <https://pypi.python.org/pypi/virtualenv>`_
and `pip <http://www.pip-installer.org/>`_.

Set-up and execution
--------------------

Create your application with a PostgreSQL DB addon::

  $ heroku login # if you haven't already
  [...]
  $ heroku create --addons heroku-postgresql:dev
  Creating [...] done, region is [...]
  Adding heroku-postgresql:dev to [...] done
  http://[...].herokuapp.com/ | git@heroku.com:[...].git
  Git remote heroku added

Then promote your database addon as your main database so it's available as
the `DATABASE_URL` configuration key (and environment variable)::

  $ heroku pg:promote `heroku addons|awk '/heroku-postgresql:dev/ {print $2}'`
  [...]

Finally push your app to heroku with::

  $ git push heroku master

This will take a while the first time you do it, downloading all the
dependencies to your heroku instance. Later pushes should run faster.

When it's finally done, you can visit it in your browser, with::

  $ heroku open

Running it locally
------------------

To run it locally, you need to create a `virtualenv` (for example in the
current directory), activate it and run::

  $ pip install -r requirements.txt

After it has finished downloading all packages needed locally, you can run it
against a locally created `FileStorage` with::

  $ pserve development.ini

If you want to run substanceD against the same heroku database you set up
above, you'll have to incorporate the remote configuration environment
locally::

  $ heroku plugins:install git://github.com/ddollar/heroku-config.git
  $ heroku config:pull -i -o

This will create a `.env` file in the current directory that'll be used by
`foreman` from the heroku toolbelt. Which you can then run with::

  $ foreman start -f Procfile.development

This is to run Pyramid (and SubstanceD) in development mode. Check out the
`Procfile.development` file to see exactly what it does. It's quite simple.

To run it locally, but in the same production mode as heroku, you can remove
the `-f Procfile.development` parameter from the command above.

This kind of execution is useful for doing maintenance tasks against the remote
data without spending execution time in your heroku instances.

How it works
============

The requirements.txt file is automatically detected by heroku as a flag for
setting up a Python environment with `virtualenv` and then running
`pip install -r requirements.txt` while building the "bundle" which will be
deployed.

The `Procfile` file is then parsed by the heroku "dyno" looking for processes
to start. This file contains instructions for calling `pserve production.ini`
and passing the configuration environment variables, which are then
"interpolated" inside "production.ini".

The two crucial variables are `DATABASE_URL`, which will be interpreted
by `zodburi` into a RelStorage ZODB configuration, and `PORT` which is where
`waitress` needs to listen for connections coming from the heroku front-end.


Technical details
=================

The contents of this package come mostly from a pristine execution of::

  $ pcreate -s substanced substancedemo

The main exceptions, and post important subjects of study for an heroku
deployment of SubstanceD are:

  * `production.ini` and `development-remote.ini`

    Interpolation of variables `%(database_url)s` and `%(http_port)s` from the
    command line to receive settings from the heroku environment.

  * `requirements.txt`

    Installing the local package and any other packages that are currently
    required in development mode from checkouts plus some version pins.

    In particular the `RelStorage` fork used provides the `zodburi`
    implementation of `postgresql://` URLs.

  * `Procfile` and `Procfile.development`

    define the command lines used to run `pserve` and pass the variables needed
    from the configuration environment to be interpolated by the `pserve`
    configuration.
