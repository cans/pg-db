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
      host: "dbserver0.domain.com"         (default: `pg_test_db_host` if defined, _see note below_)
      socket: "/run/postgres/socket"       (default: `pg_test_db_socket`)
      attr: "NOSUPERUSER,CREATEDB"         (default: `pg_test_db_default_role_attributes`)


### Specifying the databases to create

The databases to create have to be specified as a list of items as described
below:

    - name: "my_db"                    (default: _undefined_)
      extensions:                      (default: [])
        - "postgis"
      host: "dbserver0.domain.com"     (default: `pg_test_db_host` if defined _see note below_)
      socket: omit                     (default: `pg_test_db_socket` if defined _see note below_)
      users:
        - name: "alice"
          privs: ""


Note:

    Either of `socket` or `host` _should_ be defined, if neither is, then either
    of `pg_test_db_host` or `pg_test_db_socket` **must** be defined.

    This is true for items both representing roles _and_ databases.


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
- `pg_db_add`: the list of database to create (default: []);
- `pg_db_database_del`: the list of database to delete (default: []);
- `pg_db_role_add`: the list of roles to create (default: []);
- `pg_db_role_del`: the list of roles to delete (default: []);
- `pg_db_socket`: the UNIX domain socket through which connect to the
  server (default: *undefined*);
- `pg_db_system_user`: system user the cluster runs as. If set, this
  role _su_ as that user before issuing priviledged commands.


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
