# Ansible role: Postgres Backups

Installs a script to back up PostgreSQL databases. Supports backing up
multiple hosts. Manages cron entries for regular execution.

## Role variables (default values)

~~~
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
security implications of storing passwords in plaintext (if you specify then
using `pg_password`, see examples below). By default, these files will be
readable only by their owner (usually, `root`).

When removing an item from the list, remember to change to use the `state`
parameter to remove the configuration file and the cron job from the system
(see examples below).

~~~
postgres_backup_config_dir: /etc/postgres_backup
~~~

Where to store the configuration.

~~~
postgres_backup_default_output_dir: /srv/postgres_backup
~~~

Where to store the backups. May be overridden by each entry in the backup list.

~~~
postgres_backup_default_date_format: "%Y-%m-%d_%H-%M"
~~~

How to format the output directory for each backup. May be overriden by each
entry in the backup list.

## Examples

### Hourly backup of a local PostgreSQL server

~~~
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

### No automatic backups, just install the script and configuration

~~~
postgres_backup_list:
  - name: default
    pg_username: postgres
    pg_hostname: localhost
    pg_port: 5432
~~~

### Daily backup of a remote server, using a password

~~~
postgres_backup_list:
  - name: production
    pg_username: backup
    pg_password: "hunter2"
    pg_hostname: db.example.com
    pg_port: 5432
    cron:
      minute: '0'
      hour: '0'
      day: '*'
      month: '*'
      weekday: '*'
~~~

### Remove a previously defined configuration

~~~
postgres_backup_list:
  - name: production
  	state: absent
~~~
