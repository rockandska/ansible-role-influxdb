ansible-role-influxdb
=========

Ansible role to install/configure InfluxDB from InfluxDB binaries.  
Available on [Ansible Galaxy](https://galaxy.ansible.com/rockandska/influxdb)

**Travis Build :**  
[![Build Status](https://travis-ci.com/rockandska/ansible-role-influxdb.png?branch=master)](https://travis-ci.com/rockandska/ansible-role-influxdb) 

Requirements on remote hosts
------------

#### All distro

- logrotate
- python requests >= 1.0.0 ( if using users,databases,retention_policies management provide by this role )
- python influxdb >= 0.9.0  ( if using users,databases,retention_policies management provide by this role )

#### Debian / Ubuntu

- apt-transport-https
- gpg-agent
- ca-certificates

#### CentOS / RedHat
- gnupg2

Role Variables
--------------

Defaults variables are inside `defaults/main.yml`
```yaml
---
###########
# Install #
###########
influxdb_version: 1.7.4

influxdb_rpm_url: "https://dl.influxdata.com/influxdb/releases/influxdb-{{ influxdb_version }}.x86_64.rpm"
influxdb_rpm_sha256: "93caa49159e76b3338abc191c85d46d134ac67697a0c7f874c937baa38dd6a5e"
influxdb_rpm_dest_path: "/usr/src"

influxdb_deb_url: "https://dl.influxdata.com/influxdb/releases/influxdb_{{ influxdb_version }}_amd64.deb"
influxdb_deb_sha256: "5488bd8889de7d45aa09cdccc845156e2d59442871ad6899d64d797e42f2888b"
influxdb_deb_dest_path: "/usr/src"

#################
# Custom Config #
#################
influxdb_vars_files: []

influxdb_config_tpl: etc/influxdb/influxdb.conf.j2
influxdb_config: {}

influxdb_systemd_override_tpl: etc/systemd/system/influxdb.service.d/override.conf.j2
influxdb_systemd_override: {}

influxdb_custom_logrotate_tpl: etc/logrotate.d/influxdb.j2
influxdb_custom_logrotate:

############
# Api user #
############
influxdb_management_hostname:
influxdb_management_port:
influxdb_management_validate_certs:
influxdb_management_proxies:
influxdb_management_login_username:
influxdb_management_login_password:
influxdb_management_retries:
influxdb_management_ssl:
influxdb_management_timeout:
influxdb_management_udp_port:
influxdb_management_use_udp:

#########
# Users #
#########
influxdb_users_to_create: []
influxdb_users_to_delete: []

#############
# Databases #
#############
influxdb_databases_to_create: []
influxdb_databases_to_delete: []

############
# Policies #
############
influxdb_retention_policies_to_create: []

#########
# Debug #
#########
influxdb_hide_log: true
```

#### Details

- `influxdb_version`

  - should be a influxdb version available on https://dl.influxdata.com/influxdb/releases
  - old versions are not shown on download page but are available from direct links

- `influxdb_rpm_url`

  - url of the rpm package

- `influxdb_rpm_sha256`

  - sha256 of the rpm package to verify its integrity
  - Available on InfluxDB download page for the last release

- `influxdb_rpm_dest_path`

  - path where to download the rpm package

- `influxdb_deb_url`

  - url of the deb package

- `influxdb_deb_sha256`

  - sha256 of the deb package to verify its integrity
  - Available on InfluxDB download page for the last release

- `influxdb_deb_dest_path`

  - path where to download the deb package

- `influxdb_vars_files`

  - list of vars files used to override defaults variables if needed
  - if using relative path, put those files next to your playbook in `vars` directory
  - example:
    ```yaml
    influxdb_vars_files:
      - settings.yml
    ```

- `influxdb_config_tpl`

  - path to the influxdb config template
  - if you want to use your own template
    - add your template next to your playbook in a `templates` directory
    - use a different path than the default one

- `influxdb_config`

  - a dict representing the custom influxdb config to apply

  - put specials keys between double quotes (example: "[meta]")

  - If using custom config, some minimal sections need to be apply due to [influxdb/#12140](https://github.com/influxdata/influxdb/issues/12140) and are shown in the example bellow

  - examples:
    ```yaml
    influxdb_config:
    # Global config section as 'key: value'
      reporting-disabled: true
    # Minimal required sections as 'section: { key: value, key: value }'
      "[meta]":
        dir: /var/lib/influxdb/meta
      "[data]":
        dir: /var/lib/influxdb/data
        wal-dir: /var/lib/influxdb/wal
    # Other sections in format as 'section: { key: value, key: value }'
      "[http]":
        auth-enabled: true
      "[[graphite]]":
        enable: true
        batch-size: 5000
    ```

- `influxdb_systemd_override_tpl`

  - path to the influxdb systemd override template
  - if you want to use your own template
    - add your template next to your playbook in a `templates` directory
    - use a different path than the default one

- `influxdb_systemd_override`

  - a dict representing the systemd override config
  - the first level is used for the ini section
  - the second level is used for the key / value 
  - example:
    ```yaml
    influxdb_systemd_override:
      Service:
        LimitNOFILE: 30000
    # Will result into the systemd override file as:
    # [Service]
    # LimitNOFILE=30000
    ```

- `influxdb_custom_logrotate_tpl`

  - path to the influxdb custom logrotate template
  - if you want to use your own template
    - add your template next to your playbook in a `templates` directory
    - use a different path than the default one

- `influxdb_custom_logrotate`

  - a multiline string with the logrotate options for influxdb logs

  - will erase the default config

  - **/!\ Be aware that if you replace the default logrotate config by a custom one, the configuration applied will persist even if you unset this variable**

  - example:
    ```yaml
    influxdb_custom_logrotate: |
      weekly
      missingok
      rotate 40
      compress
      notifempty
    # Will result into the logrotate config file as:
    # /var/log/influxdb/*.log {
    #   weekly
    #   missingok
    #   rotate 40
    #   compress
    #   notifempty
    # }
    ```

- `influxdb_management_hostname`
  - default from influxdb_{user,policy,retention_policy} module value
  - hostname for API calls

- `influxdb_management_port`
  - default from influxdb_{user,policy,retention_policy} module value
  - http port for API calls

- `influxdb_management_validate_certs`
  - default from influxdb_{user,policy,retention_policy} module value
  - do we need to validate the SSL certificate for API calls

- `influxdb_management_proxies`
  - default from influxdb_{user,policy,retention_policy} module value
  - HTTP(S) proxy to use for API calls

- `influxdb_management_login_username`
  - username to use for API calls
  - **mandatory if `influxdb_config['[http]']['auth-enabled']` is set**
  - This user will be create only once, and need to be a **none already existing user**
  - If you want to change the user afterward its creation, use an already existing `admin` user.

- `influxdb_management_login_password`

  - password to use for API calls
  - **mandatory if `influxdb_config['[http]']['auth-enabled']` is set**
  - If you want to change the password afterward the `admin` user creation:
    - Change the password through the CLI before changing it here
    - Alternatively, you could change it into `influxdb_users_to_create`, but be aware that it will throw an error if you have remaining users who need to be checked but should works if you relaunch your playbook a second time.

- `influxdb_management_retries`

  - default from influxdb_{user,policy,retention_policy} module value
  - number of retries for API calls

- `influxdb_management_ssl`

  - default from influxdb_{user,policy,retention_policy} module value
  - use SSL for API calls

- `influxdb_management_timeout`
  - default from influxdb_{user,policy,retention_policy} module value
  - timeout for API calls

- `influxdb_management_udp_port`
  - default from influxdb_{user,policy,retention_policy} module value
  - UDP port to use for API calls

- `influxdb_management_use_udp`
  - default from influxdb_{user,policy,retention_policy} module value
  - do we need to use UDP for API calls

- `influxdb_users_to_create`

  - list of dict for users creation

  - refer to [ansible doc](https://docs.ansible.com/ansible/latest/modules/influxdb_user_module.html) for mandatory options and version compatibility

  - **grant privileges are provided by a custom library provided with this role and update from this [PR](https://github.com/ansible/ansible/pull/46216)**

  - **the absence of `grants` key will remove all `GRANTS` for the user specified**

  - Example:
    ```yaml
    influxdb_users_to_create:
      - user_name: test
        user_password: test
        grants:
          - database: db_test
            privilege: READ  
      - user_name: test2
        user_password: test2
        admin: true  
    ```

- `influxdb_users_to_delete`
  - list of users to delete
  - Example:
    ```yaml
    influxdb_users_to_delete:
      - test
    ```

- `influxdb_retention_policies_to_create`
  - list of retention policies to create
  - refer to [ansible doc](https://docs.ansible.com/ansible/latest/modules/influxdb_retention_policy_module.html) for mandatory options and version compatibility
  - example:

    ```yaml
    influxdb_retention_policies_to_create:
      - database_name: db_test
        policy_name: test
        duration: 1h
        replication: 1
    ```

- `influxdb_databases_to_create`

    - list of databases to create
    - examples:

  ```yaml
  influxdb_databases_to_create:
    - db_test
  ```

- `influxdb_databases_to_delete`


    - list of databases to delete

    - examples:

  ```yaml
  influxdb_databases_to_delete:
    - db_test
  ```

- `influxdb_retention_policies_to_create`


    - list of dict retention policies parameters

    - **deletion of policies are not supported yet**

    - examples:

  ```yaml
  influxdb_retention_policies_to_create:
    - database_name: db_test
      policy_name: test
      duration: 1h
      replication: 1  
  ```

- `influxdb_hide_log`

  - default: true
  - don't show the log for api calls to avoid leak of sensible informations
  - set it to false for debug

Example Playbook
----------------

```yaml
- hosts: influxdb
  roles:
    - rockandska.influxdb
```

Local Testing
-------------

The preferred way of locally testing the role is to use Docker and [molecule](https://github.com/metacloud/molecule) (v2.x). You will have to install Docker on your system. See "Get started" for a Docker package suitable to for your system.
We are using tox to simplify process of testing on multiple ansible versions. To install tox execute:
```sh
$ sudo pip install tox
# or
$ pip install --user tox
```

To run tests on all ansible versions (WARNING: this can take some time)
```sh
$ tox
```
To run a custom molecule command on custom environment with only default test scenario:
```sh
$ tox -e py27-ansible25 -- molecule test -s default
```
For more information about molecule go to their [docs](http://molecule.readthedocs.io/en/latest/).

If you would like to run tests on remote docker host just specify `DOCKER_HOST` variable before running tox tests.

License
-------

BSD
