# Rockabox WSGI App Container
This is a role for installing a wsgi application container. It makes the
following assumptions about your application:

* It has a [wsgi](http://wsgi.readthedocs.org/en/latest/) file, see below
* You have installed Nginx in another role prior to running this one \*
* You have a public key generated and stored at `~/.ssh/id_rsa.pub`
(see [this guide](https://help.github.com/articles/generating-ssh-keys/) for help)**

\* e.g - [Stouts Nginx](https://galaxy.ansible.com/list#/roles/854)

\*\* Unless you have specified another public key in `wsgi_ssh_key`

## Role variables:

### Required
* `wsgi_project_name` (The name of your project, letters and underscores only)
* `wsgi_wsgi_path` (the relative python path to your application's wsgi file)

## Overidable
Loads! take a look in defaults/main.yml for the full list

Once you have this role on your server, all you need to do is put your
application at `wsgi_project_name@host.com:current` ...and you're good to go.

To restart your application you can `sudo supervisorctl restart wsgi_project_name`
you can ssh into your box as your application with\* `wsgi_project_name@host.com`

\* Providing you did not set `wsgi_ssh: no`

The following environmental variables are made available to your application
and your ssh session:

    PREFIX_APP_PATH (The location of your project)
    PREFIX_APP_NAME (The name of your project)
    PREFIX_RELEASE_DIR (The location of your projects releases)
    PREFIX_VENV (The location of your projects virtual environment)
    PREFIX_LOG_DIR (The location of your projects log directory)
    PREFIX_PID (The location of your projects pid file)
    PREFIX_STATIC_DIR (The location of your static files directory, if you have one)
    PREFIX_MEDIA_DIR (The location of your media files directory, if you have one)

Where `PREFIX` is wsgi\_project\_name in upper case, you can override this
with `wsgi_env_prefix`

Also any databases you have defined in your application will be available as
environmental variables using the following format:

    PREFIX_DB_DBID_NAME (The name of your database)
    PREFIX_DB_DBID_USER (The username for your database)
    PREFIX_DB_DBID_PASSWORD (The password for your database)
    PREFIX_DB_DBID_HOST (The host for your database)
    PREFIX_DB_DBID_PORT (The port your database runs on)

Where DBID is the upper-cased id of your database, and PREFIX is as above.

## Pil/Pillow support

Python imaging libraries (PIL / Pillow) depend on native libraries to support
some image formats or to provide additional functionality, like JPEG or FreeType
libraries. Those dependencies can be automatically installed using `wsgi_enable_python_image: yes`

## Deployment
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

## Application Cron tasks

You can add cron tasks to your wsgi app using the `wsgi_cron_tasks` variable,
an example is as follows:

    wsgi_cron_tasks:

        name: Rotate the logs
        user: root
        command: "/usr/sbin/logrotate -f /etc/logrotate.d/log_conf"
        frequency:
            minute: "*/5"
        logfile: "/dev/null"

All of your environmental variables are made available to the process running
the cron, the command will look something like this:

    /bin/bash -c "source ~/.bash_profile && {{ command }} &>> {{ logfile }}

Note: you can set day, hour, minute or month in frequency, values not set
default to '\*'

## SSl / Listening on different ports
By default, port 80 will be listened on without SSL, to add SSL, add port 443
like so:

    wsgi_nginx_http_ports:

        - port: 80
          ssl: no

        - port: 443
          ssl: yes

And make sure you also properly override the other wsgi_ssl vars found in
defaults/main.yml, one other thing to remember is, you will need to take care
of copying your ssl certificates/keyfiles to your server youerself, this might
be done in your play like so:

  pre_tasks:

    name: Make sure the folder for the crt file exists
    file: path='{{ wsgi_ssl_crt_file|dirname }}'
          state=directory mode=700 owner=root group=root

    name: Make sure the folder fo the key certificate exists
    file: path='{{ wsgi_ssl_key_file|dirname }}'
          state=directory mode=700 owner=root group=root

    name: copy the ssl crt file across
    copy: src='../../ssl/certificate.chained.crt' dest='{{ wsgi_ssl_crt_file }}'
          mode=1130 owner=root group=root

    name: copy the ssl key file across
    copy: src='../../ssl/certificate.key' dest='{{ wsgi_ssl_key_file }}'
          mode=1130 owner=root group=root


## Multiple wsgi apps in one play

You can have multiple wsgi apps in one play, by using the `vars_file` var:

    roles:

      - Rockabox.wsgi_app_container
        vars_file: "/path/to/vars/projectA.yml"

      - Rockabox.wsgi_app_container
        vars_file: "../../vars/projectB.yml"

The path needs to be relative to the playbook, or absolute, as described in
the [ansible docs](http://docs.ansible.com/include_vars_module.html#options)

## Future Work

- Release fabric scripts for deployment
