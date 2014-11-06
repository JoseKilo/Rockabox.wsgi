This is a role for installing a django application container, It makes the
following assumptions about your application:

    - It has a wsgi file, see below
    - It has gunicorn installed*

Role variables:
---------------

Required
~~~~~~~~

    - django_project_name (The name of your project, letters underscores only)
    - django_wsgi_path (the relative python path to your applications wsgi file)

Overidable
~~~~~~~~~~
Loads! take a look in defaults/main.yml for the full list

Once you have this role on your server, all you need to do is put your
application at:

    django_project_name@host.com:current

...and your good to go.

To restart your application you can:

    sudo supervisorctl restart django_project_name

you can ssh into your box as your application with:

    django_project_name@host.com

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

Where `PREFIX` is django_project_name in upper case, you can override this
with `django_env_prefix`

Also any databases you have defined in your application will be available as
environmental variables using the following format:

    PREFIX_DB_DBNAME_NAME (The name of your database)
    PREFIX_DB_DBNAME_USER (The username for your database)
    PREFIX_DB_DBNAME_PASSWORD (The password for your database)
    PREFIX_DB_DBNAME_HOST (The host for your database)
    PREFIX_DB_DBNAME_PORT (The port your database runs on)

Where DBNAME is the upper-cased name of your database, and PREFIX is as above.
