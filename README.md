cans.pg-db
==========

[![Build Status](https://travis-ci.org/cans/pg-db.svg?branch=master)](https://travis-ci.org/cans/pg-db)

Ansible Role to create roles, databases and grant rights on the latter
to the former, on a PostgreSQL cluster


### Specifying the roles to create

The roles to create have to be specified as a list of items as described
below:

    - login: "me, myself, and I"           (default: _undefined_)
      password: "{{variable_in_a_vault}}"  (default: _undefined_)
      host: "dbserver0.domain.com"         (default: `pg_test_db_host` if defined, see note below)
      socket: "/run/postgres/socket"       (default: `pg_test_db_socket`)
      attr: "NOSUPERUSER,CREATEDB"         (default: `pg_test_db_default_role_attributes`)

Where:

- `attr`: lists the attributes the user should have (_cf._
  [CREATE ROLE](https://www.postgresql.org/docs/current/static/sql-createrole.html));
- `host`: the hostname through which connect to the database cluster
  (implies use of TCP/IP connection, mutually exclusive with `socket`);
- `login`: the name to give to the new user;
- `password`: the password to set for the new user;
- `socket`: the path to a Unix Domain through which connect to the
  database cluster (mutually exclusive with `host`);

### Specifying the databases to create

The databases to create have to be specified as a list of items as described
below:

    - name: "my_db"                    (default: _undefined_)
      extensions:                      (default: [])
        - "postgis"
      host: "dbserver0.domain.com"     (default: `pg_db_host` if defined, see note below)
      socket: omit                     (default: `pg_db_socket` if defined, see note below)
      owner: "alice"
      users:
        - name: "alice"
          privs: "ALL"

Where:

- `extensions`: lists the names of the postgres extension to install in
  the database (default: *[]*);
- `host`: the hostname through which connect to the database cluster
  (implies use of TCP/IP connection, mutually exclusive with `socket`);
- `name`: is the name of the database to create (*mandatory*);
- `owner`: is the name of the user that should own the database
  (default: *omit*, meaning *pg_db_admin_user* will own the database);
- `socket`: the path to a Unix Domain through which connect to the
  database cluster (mutually exclusive with `host`);
- `users`: a list of users to grant some privileges to on the created
  database (default: *[]*). Each user must be given a `login` and
  a list of privileges via the `privs` variable;

Users, whether specified in the `owner` or in the `users` list are
assumed to exist *a priori*

Notes:

- Either of `socket` or `host` *should* be defined, if neither is, then either
  of `pg_db_host` or `pg_db_socket` **must** be defined.
  This is true for both items representing roles _and_ databases;
- This role will create users, before creating databases to be able to
  set ownership and privileges properly;


### Regarding tests and authentication

This role comes with what tries to be a thorough test suite (admittedly
incomplete as of yet).

On challenge is to make sure authentication works depending on the very
many scenarios you could face:

- playbook ran locally connecting to a local Postgres cluster;
- playbook ran locally connecting to a remote Postgres cluster;
- playbook ran on a remote target and connecting to database local to
  the target host;
- playbook ran on a remote target used to bounce to yet another host
  where the Postgres cluster is hosted;

Some of that test suit is ran on the popular [TravisCI](travis-ci.org)
online continuous integration service. Sadly, their build image come
with very lax security set-up. This may hide issues with ill-handled
scenarios amongst those listed above and the different connection and
authentication options they offer.

It is thus not recommanded to trust those tests outcome and run the
tests on set-up of your own. For its development, this role's tests
were typically run against a Debian distro inside a VM. The only
modifications required being the following:

- On the VM add a sudoer user (e.g. `ansible`) that does not require
  a password (basically what you get e.g. if you set-up the VM with
  vagrant).
- Inside the `tests/` add a `inventory.local` file that points to the
  VM, as follows:

       [servers]
       192.168.X.X ansible_user=<sudoer user>

  Where `<sudoer user>` is to be substituted for the name of the user
  you added at the previous step;

You can then launch the `tests/run.sh` script to run the tests.


Requirements
------------

This role has the dependencies of Ansible's PostgreSQL related modules:

- [`postgresql_db`](http://docs.ansible.com/ansible/postgresql_db_module.html)
- [`postgresql_ext`](http://docs.ansible.com/ansible/postgresql_ext_module.html)
- [`postgresql_user`](http://docs.ansible.com/ansible/postgresql_user_module.html)
- [`postgresql_privs`](http://docs.ansible.com/ansible/postgresql_privs_module.html)

Which basically means that you need psycopg2 installed on the target host. The
`pg_db_apt_packages` variable found in `vars/main.yml` provides a list of
packages you need to install for this role to work.


Role Variables
--------------

All the variables in this role are namespaced with the prefix `pg_db`

- `pg_db_admin_db`: name of cluster's administration database user
  (default: "postgres")
- `pg_db_admin_user`: login name of the postgresql super (admin)
  user (default: "postgres")
- `pg_db_apt_packages`: the list of Debian packages to install on
  the target host(s) for this role to work.
- `pg_db_change_password`: whether to change the user's password if needed
  see comment in the
  [`postgresql_user` documentation](http://docs.ansible.com/ansible/postgresql_user_module.html)
  (default: no)
- `pg_db_encoding`: (default: "UTF-8")
- `pg_db_encrypt_password`: (default: yes)
- `pg_db_extensions`:  (default: []);
- `pg_db_host`: the default database server host to connect to (default: _omit_);
- `pg_db_database_add`: the list of database to create (default: []);
- `pg_db_database_del`: the list of database to delete (default: []);
- `pg_db_socket`: the UNIX domain socket through which connect to the
  server (default: *undefined*);
- `pg_db_system_user`: system user the cluster runs as. If set, this
  role will become (_su_ as) that user before issuing priviledged commands.
- `pg_db_user_add`: the list of roles to create (default: []);
- `pg_db_user_del`: the list of roles to delete (default: []);


Dependencies
------------

This role has no dependency.


Example Playbook
----------------

The following example will connect to the postgreSQL cluster on the local
machine, through a unix socket, and create a role `alice` and a database
named `test_db` and owned by alice.


    - hosts: 127.0.0.1
      connection: local
      vars_files:
        - vars/vault.yml  # To store credentials: pg_db_admin_login, pg_db_admin_password, _etc._

      vars:
        pg_db_user_add:
          - name: alice
            password: "{{alices_password}}"
            socket: "/run/postgresql"
        pg_db_database_add:
          - name: test_db
            socket: "/run/postgresql"
            owner: alice
        pg_db_admin_user: "postgres"
        pg_db_admin_password: "{{db_admins_password}}"

      roles:
         - role: cans.pg-db


License
-------

GPLv2


Author Information
------------------

Copyright © 2017, Nicolas CANIART
