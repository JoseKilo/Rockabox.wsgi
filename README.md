# Rockabox WSGI
This is a role for installing a wsgi application container. It makes the
following assumptions about your application:

* It has a [wsgi](http://wsgi.readthedocs.org/en/latest/) file, see below

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

you can ssh into your box as your application with*:

    wsgi_project_name@host.com

\* Providing you did not set `wsgi_ssh: no`

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

    PREFIX_DB_DBNAME_NAME (The name of your database)
    PREFIX_DB_DBNAME_USER (The username for your database)
    PREFIX_DB_DBNAME_PASSWORD (The password for your database)
    PREFIX_DB_DBNAME_HOST (The host for your database)
    PREFIX_DB_DBNAME_PORT (The port your database runs on)

Where DBNAME is the upper-cased name of your database, and PREFIX is as above.


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


Application Cron tasks
~~~~~~~~~~~~~~~~~~~~~~
You can add cron tasks to your wsgi app using the `wsgi_cron_tasks` variable,
an example is as follows:

    wsgi_cron_tasks:

        name: Rotate the logs
        user: root
        command: "/usr/sbin/logrotate -f /etc/logrotate.d/log_conf"
        frequency: "*/5"
        logfile: "/dev/null"

All of your environmental variables are made available to the process running
the cron, the command will look something like this:

    /bin/bash -c "source ~/.bash_profile && {{ command }} &>> {{ logfile }}

Note: the `frequency` parameter is passed through to `minute`, so ensure you
use the correct syntax e.g:

    frequency: */5 # Runs every 5 minutes
    frequency: 5 # Runs at 5 minutes past every hour

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
