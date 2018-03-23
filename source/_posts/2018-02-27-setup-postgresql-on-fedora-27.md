title: "Setup PostgreSQL on Fedora 27"
date: 2018-02-27 14:52:27
tags:
- PostgreSQL
- Fedora
---
# Installation

## Installing the Version Contained in Fedora's Repo

PostgreSQL can be installed with *yum*, the package system of Fedora.

```
$ yum install postgresql-server postgresql-contrib
```

This will install the version contained in Fedora's repo (currently 9.6.7-1).

## Installing a Specified Version

To install a specified repo, find an RPM repo package from [https://yum.postgresql.org/repopackages.php](https://yum.postgresql.org/repopackages.php).

For example, to install PostgreSQL 9.6:

```
$ rpm -Uvh [https://download.postgresql.org/pub/repos/yum/9.6/fedora/fedora-27-x86_64/pgdg-fedora96-9.6-4.noarch.rpm](https://download.postgresql.org/pub/repos/yum/9.6/fedora/fedora-27-x86_64/pgdg-fedora96-9.6-4.noarch.rpm)
$ yum install postgresql96-server postgresql96-contrib
```

## Post-Installation Setup

After yum finish installation, initialize PostgreSQL with

```
$ postgresql-setup initdb
```

This creates a new PostgreSQL database cluster.

Start and enable PostgreSQL service:

```
$ systemctl start postgresql
$ systemctl enable postgresql
```

# Creating Roles and Databases

PostgreSQL uses "roles" for authentication, which is similar to Unix users and groups. A Unix user "postgres" will be created upon installation. To log into PostgreSQL with this identity:

```
$ sudo -i -u postgres
<type password>
$ psql
```

Command `psql` uses Unix socket file to connect to the PostgreSQL server by default. `peer` is the default authentication method for Unix socket file connections, which looks up the PostgreSQL role that is related to the Unix user name.

After logging into PostgreSQL, use `\q` to quit.

With the "postgres" Unix account, you can create PostgreSQL roles and databases. To create a new role:

```
$ createuser --interactive
```

You will be asked for the user name and other information about the role to create.

To create a database for the role:

```
$ createdb <db name>
```

In order to log into PostgreSQL with your own Unix account directly, you should create a PostgreSQL role and a database that have the same name as your Unix account.

## Testing

With your own Unix account, log into PostgreSQL with:

```
$ psql
```

This will use the PostgreSQL role and connect to the database that have the same name as your account. To view the connection info, use `\conninfo` in the PostgreSQL prompt.

```
postgres=# \conninfo
You are connected to database "<your Unix account>" as user "<your Unix account>" via socket in "/var/run/postgresql" at port "5432".
```

# Updating Authentication Method

By default PostgreSQL uses `peer` authentication for connections through Unix socket file, and `ident` for local TCP/IP connections. This can be modified by editing `/var/lib/pgsql/data/pg_hba.conf`.

**Update (Mar 23, 2018)**: The path to `pg_hba.conf` may vary. You can ask PostgreSQL for the real path with `SHOW hba_file;`.

The default options are around the end of the file:

```
# TYPE  DATABASE  USER  ADDRESS       METHOD

# "local" is for Unix domain socket connections only
local   all       all                 peer
# IPv4 local connections:
host    all       all   127.0.0.1/32  ident
# IPv6 local connections:
host    all       all   ::1/128       ident
```

For example, to force password when logging in through TCP/IP connections, replace `ident` with `md5`. A list of supported authentication methods can be found from [https://www.postgresql.org/docs/9.6/static/auth-methods.html](https://www.postgresql.org/docs/9.6/static/auth-methods.html).
