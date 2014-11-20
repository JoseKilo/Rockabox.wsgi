# Django WSGI
This is a role for installing a wsgi application container, It makes the
following assumptions about your application:

* It has a [wsgi](http://wsgi.readthedocs.org/en/latest/) file, see below
* It has [gunicorn](http://gunicorn.org/) installed*

* gunicorn is required In your projects virtualenv rather than globally, to
allow for multiple projects on one server

## Role variables:

### Required
* wsgi_project_name (The name of your project, letters underscores only)
* wsgi_wsgi_path (the relative python path to your applications wsgi file)

## Overidable
Loads! take a look in defaults/main.yml for the full list

Once you have this role on your server, all you need to do is put your
application at:

    wsgi_project_name@host.com:current

...and your good to go.

To restart your application you can:

    sudo supervisorctl restart wsgi_project_name

you can ssh into your box as your application with:

    wsgi_project_name@host.com

The following environmental variables are made available to your application
and your ssh session:

    PREFIX_APP_PATH (The location of your project)
    PREFIX_APP_NAME (The name of your project)
    PREFIX_APP_RELEASE_DIR (The location of your projects releases)
    PREFIX_APP_VENV (The location of your projects virtual environment)
    PREFIX_APP_LOG_DIR (The location of your projects log directory)
    PREFIX_APP_PID (The location of your projects pid file)
    PREFIX_APP_STATIC_DIR (The location of your static files directory, if you have one)
    PREFIX_APP_MEDIA_DIR (The location of your media files directory, if you have one)

Where `PREFIX` is wsgi_project_name in upper case, you can override this
with `wsgi_env_prefix`

Also any databases you have defined in your application will be available as
environmental variables using the following format:

    PREFIX_DB_DBID_NAME (The name of your database)
    PREFIX_DB_DBID_USER (The username for your database)
    PREFIX_DB_DBID_PASSWORD (The password for your database)
    PREFIX_DB_DBID_HOST (The host for your database)
    PREFIX_DB_DBID_PORT (The port your database runs on)

Where DBID is the upper-cased id of your database, and PREFIX is as above.


Deployment
----------
To deploy to this container, ensure your project is available in

    $PREFIX_APP_RELEASE_DIR/current

And ensure your projects dependencies are installed into

    $PREFIX_APP_VENV

Once deployed, restart your application with

    sudo service $PREFIX_APP_NAME restart

Ensure your media is stored in `$PREFIX_APP_MEDIA_DIR`, and your
static files in `$PREFIX_APP_STATIC_DIR`

To see stdout and stderr from your application, see:

    $PREFIX_APP_LOG_DIR/supervisor.log


Multiple wsgi apps in one play
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
You can have multiple wsgi apps in one play, by using the `vars_file` var:

  roles:
    - {role: Rockabox.wsgi, vars_file: "/path/to/vars/projectA.yml"}
    - {role: Rockabox.wsgi, vars_file: "../../vars/projectB.yml"}

The path needs to be relative to the playbook, or absolute, as described in
the [ansible docs](http://docs.ansible.com/include_vars_module.html#options)

Future Work
-----------
- Release fabric scripts for deployment
