PostgreSQL for OpenShift - Docker images
========================================

This repository contains Dockerfiles for PostgreSQL images for OpenShift.
Users can choose between RHEL and CentOS based images.


Versions
---------------
PostgreSQL versions currently provided are:
* postgresql-9.2

RHEL versions currently supported are:
* RHEL7

CentOS versions currently supported are:
* CentOS7


Installation
----------------------
Choose either the CentOS7 or RHEL7 based image:

*  **RHEL7 based image**

    To build a RHEL7 based image, you need to run Docker build on a properly
    subscribed RHEL machine.

    ```
    $ git clone https://github.com/openshift/postgresql.git
    $ cd postgresql
    $ make build TARGET=rhel7 VERSION=9.2
    ```

*  **CentOS7 based image**

    This image is available on DockerHub. To download it run:

    ```
    $ docker pull openshift/postgresql-92-centos7
    ```

    To build a PostgreSQL image from scratch run:

    ```
    $ git clone https://github.com/openshift/postgresql.git
    $ cd postgresql
    $ make build VERSION=9.2
    ```

**Notice: By omitting the `VERSION` parameter, the build/test action will be performed
on all provided versions of PostgreSQL. Since we are currently providing only version `9.2`,
you can omit this parameter.**


Environment variables and volumes
----------------------------------

The image recognizes the following environment variables that you can set during
initialization by passing `-e VAR=VALUE` to the Docker run command.

|    Variable name             |    Description                                 |
| :--------------------------- | ---------------------------------------------- |
|  `POSTGRESQL_USER`           | User name for PostgreSQL account to be created |
|  `POSTGRESQL_PASSWORD`       | Password for the user account                  |
|  `POSTGRESQL_DATABASE`       | Database name                                  |
|  `POSTGRESQL_ADMIN_PASSWORD` | Password for the `postgres` admin account (optional)     |

The following environment variables influence the PostgreSQL configuration file. They are all optional.

|    Variable name              |    Description                                                          |    Default
| :---------------------------- | ----------------------------------------------------------------------- | -------------------------------
|  `POSTGRESQL_MAX_CONNECTIONS` | The maximum number of client connections allowed. This also sets the maximum number of prepared transactions. |  100
|  `POSTGRESQL_SHARED_BUFFERS`  | Sets how much memory is dedicated to PostgreSQL to use for caching data |  32M

You can also set the following mount points by passing the `-v /host:/container` flag to Docker.

|  Volume mount point      | Description                           |
| :----------------------- | ------------------------------------- |
|  `/var/lib/pgsql/data`   | PostgreSQL database cluster directory |

**Notice: When mouting a directory from the host into the container, ensure that the mounted
directory has the appropriate permissions and that the owner and group of the directory
matches the user UID or name which is running inside the container.**

Usage
----------------------

For this, we will assume that you are using the `openshift/postgresql-92-centos7` image.
If you want to set only the mandatory environment variables and not store the database
in a host directory, execute the following command:

```
$ docker run -d --name postgresql_database -e POSTGRESQL_USER=user -e POSTGRESQL_PASSWORD=pass -e POSTGRESQL_DATABASE=db -p 5432:5432 openshift/postgresql-92-centos7
```

This will create a container named `postgresql_database` running PostgreSQL with
database `db` and user with credentials `user:pass`. Port 5432 will be exposed
and mapped to the host. If you want your database to be persistent across container
executions, also add a `-v /host/db/path:/var/lib/pgsql/data` argument. This will be
the PostgreSQL database cluster directory.

If the database cluster directory is not initialized, the entrypoint script will
first run [`initdb`](http://www.postgresql.org/docs/9.2/static/app-initdb.html)
and setup necessary database users and passwords. After the database is initialized,
or if it was already present, [`postgres`](http://www.postgresql.org/docs/9.2/static/app-postgres.html)
is executed and will run as PID 1. You can stop the detached container by running
`docker stop postgresql_database`.


PostgreSQL admin account
------------------------
The admin account `postgres` has no password set by default, only allowing local
connections.  You can set it by setting the `POSTGRESQL_ADMIN_PASSWORD` environment
variable when initializing your container. This will allow you to login to the
`postgres` account remotely. Local connections will still not require a password.


Changing passwords
------------------

When you use a persistent volume to store your data, you may want to update
passwords of an existing database. Since passwords are part of the image
configuration, you are not supposed to change any database password through
SQL statements or any database client whatsoever.

The only supported method to change passwords for the database user (defined
by `POSTGRESQL_USER`) and admin ('postgres') is by changing the environment
variables `POSTGRESQL_PASSWORD` and `POSTGRESQL_ADMIN_PASSWORD`, respectively.


Usage on Atomic host
---------------------------------
Systems derived from projectatomic.io usually include the `atomic` command that is
used to run containers besides other things.

To install a new service `postgresql1` based on this image on such a system, run:

```
$ atomic install -n postgresql1 --opt2='-e POSTGRESQL_USER=user -e POSTGRESQL_PASSWORD=secretpass -e POSTGRESQL_DATABASE=db1 -p 5432:5432' openshift/postgresql-92-centos7
```

Then to run the service, use the standard `systemctl` call:

```
$ systemctl start postgresql1.service
```

In order to work with that service, you may either connect to exposed port 5432 or run this command to connect locally:
```
$ atomic run -n postgresql1 openshift/postgresql-92-centos7 bash -c 'psql'
```

To stop and uninstall the postgresql1 service, run:

```
$ systemctl stop postgresql1.service
$ atomic uninstall -n postgresql1 openshift/postgresql-92-centos7
```


Test
---------------------------------

This repository also provides a test framework, which checks basic functionality
of the PostgreSQL image.

Users can choose between testing PostgreSQL based on a RHEL or CentOS image.

*  **RHEL based image**

    To test a RHEL7 based PostgreSQL image, you need to run the test on a properly
    subscribed RHEL machine.

    ```
    $ cd postgresql
    $ make test TARGET=rhel7 VERSION=9.2
    ```

*  **CentOS based image**

    ```
    $ cd postgresql
    $ make test VERSION=9.2
    ```

**Notice: By omitting the `VERSION` parameter, the build/test action will be performed
on all provided versions of PostgreSQL. Since we are currently providing only version `9.2`,
you can omit this parameter.**
