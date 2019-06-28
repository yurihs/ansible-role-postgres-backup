# Ansible Role: Postgres Backups

[![Ansible Galaxy Badge](https://img.shields.io/ansible/role/41600.svg)](https://galaxy.ansible.com/yurihs/postgres_backup)

- Installs a script to back up PostgreSQL databases.
- Supports backing up multiple hosts.
- Manages cron entries for regular execution.


## What is backed up?

- Global objects (roles and tablespaces), compressed using gzip. `pg_dumpall --globals-only | gzip`
- All readable databases, to separate files, using the "custom" PostgreSQL format. `pg_dump --format=custom database`


## Role variables (default values)

~~~yaml
postgres_backup_list:
  - name: default
    pg_username: postgres
    pg_hostname: localhost
    pg_port: 5432
    cron:
      minute: '0'
      hour: '*'
      day: '*'
      month: '*'
      weekday: '*'
~~~

Which PostgreSQL instances to backup (connection info), and (optionally) how
often.

These entries are stored in configuration files, so you must consider the
security implications of storing passwords in plaintext (if you specify them
using `pg_password`, see examples below). By default, these files will be
readable only by their owner (usually `root`).

When removing an item from the list, remember to change to use the `state`
parameter to remove the configuration file and the cron job from the system
(see examples below).

~~~yaml
postgres_backup_config_dir: /etc/postgres_backup
~~~

Where to store the configuration.

~~~yaml
postgres_backup_default_output_dir: /srv/postgres_backup
~~~

Where to store the backups. Each entry will have its own directory inside
here. This variable may be overridden by each entry in the backup list.

~~~yaml
postgres_backup_default_date_format: "%Y-%m-%d_%H-%M"
~~~

How to format the output directory for each backup. May be overriden by each
entry in the backup list.

## Examples

### Hourly backup of a local PostgreSQL server

~~~yaml
postgres_backup_list:
  - name: default
    pg_username: postgres
    pg_hostname: localhost
    pg_port: 5432
    cron:
      minute: '0'
      hour: '*'
      day: '*'
      month: '*'
      weekday: '*'
~~~

This configuration will result in the following structure:

~~~
/srv/
  postgres_backup/
    default/
      2019-01-01_00-00/
        globals.sql.gz
        database-a.custom
        database-b.custom
      2019-01-01_01-00/
        globals.sql.gz
        database-a.custom
        database-b.custom
~~~

### No automatic backups, just install the script and configuration

~~~yaml
postgres_backup_list:
  - name: default
    pg_username: postgres
    pg_hostname: localhost
    pg_port: 5432
~~~

### Daily backup of a remote server, using a password, and output to a different directory

~~~yaml
postgres_backup_list:
  - name: production
    pg_username: backup
    pg_password: "hunter2"
    pg_hostname: db.example.com
    pg_port: 5432
    output_dir: /opt/prod_db_bak
    cron:
      minute: '0'
      hour: '0'
      day: '*'
      month: '*'
      weekday: '*'
~~~

This configuration will result in the following structure:

~~~
/opt/
  prod_db_bak/
    2019-01-01_00-00/
      globals.sql.gz
      users.custom
      posts.custom
    2019-01-02_00-00/
      globals.sql.gz
      users.custom
      posts.custom
~~~

### Remove a previously defined configuration

~~~yaml
postgres_backup_list:
  - name: production
    state: absent
~~~
