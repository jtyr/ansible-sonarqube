sonarqube
=========

Ansible role which helps to intall and configure SonarQube.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
- name: SonarQube with PostgreSQL
  hosts: myhost
  roles:
    - postgresql
    - sonarqube

- name: SonarQube with PostgreSQL with custom configuration
  hosts: myhost
    # Set the custom user and password
    sonarqube_db_user: sonaruser
    sonarqube_db_password: S0narP4ssword
    # Add some more Sonar configuration
    sonarqube_config__custom:
      # Change the default port to 8080
      sonar.web.port: 8080
      # Change the Elasticsearch host
      sonar.search.host: 192.168.1.234
  roles:
    - postgresql
    - sonarqube

- name: SonarQube with MySQL
  hosts: myhost
  vars:
    # Tell it to use MySQL instead of the default PostgreSQL
    sonarqube_db_engine: mysql
  roles:
    - mysql
    - sonarqube
```


Role variables
--------------

```
# YUM repo URL
sonarqube_yumrepo_url: http://downloads.sourceforge.net/project/sonar-pkg/rpm

# Additional YUM repo pararms
sonarqube_yumrepo_params: {}

# Package to be installed (explicit version can be specified here)
sonarqube_pkg: sonar

# Packages required for the PostgreSQL DB creation
sonarqube_db_pgsql_pkgs:
  - python-psycopg2

# Packages required for the MySQL DB creation
sonarqube_db_mysql_pkgs:
  - MySQL-python

# Default list of extra packages
sonarqube_deps_pkgs__default:
  - java

# Custom list of extra packages
sonarqube_deps_pkgs__custom: []

# Final list of extra packages
sonarqube_deps_pkgs: "{{
  (
    sonarqube_db_pgsql_pkgs
      if sonarqube_db_engine == 'postgresql'
      else
    sonarqube_db_mysql_pkgs
      if sonarqube_db_engine == 'mysql'
      else
    []
  ) +
  sonarqube_deps_pkgs__default +
  sonarqube_deps_pkgs__custom }}"

# Name of the service
sonarqube_service: sonar

# Config directory
sonarqube_conf_dir: /opt/sonar/conf

# DB engine [postgresql|mysql]
sonarqube_db_engine: postgresql

# User used to create DB and user
sonarqube_db_login_user: "{{
  'postgres'
    if sonarqube_db_engine == 'postgresql'
    else
  'root' }}"

# Password used to create DB and user
sonarqube_db_login_password: "{{
  'postgres'
     if sonarqube_db_engine == 'postgresql'
     else
  None }}"

# DB server configuration options
sonarqube_db_host: localhost
sonarqube_db_port: "{{
  5432
    if sonarqube_db_engine == 'postgresql'
    else
  3306 }}"
sonarqube_db_name: sonar
sonarqube_db_user: sonar
sonarqube_db_password: sonar

# User used to create DB and user
sonarqube_db_jdbc_params: "{{
  '?useUnicode=true&characterEncoding=utf8&useConfigs=maxPerformance&rewriteBatchedStatements=true'
    if sonarqube_db_engine == 'mysql'
    else
  '' }}"


# Default options of the DB configuration
sonar_conf_db__default:
  sonar.jdbc.username: "{{ sonarqube_db_user }}"
  sonar.jdbc.password: "{{ sonarqube_db_password }}"
  sonar.jdbc.url: jdbc:{{ sonarqube_db_engine }}://{{ sonarqube_db_host }}:{{ sonarqube_db_port }}/{{ sonarqube_db_name }}{{ sonarqube_db_jdbc_params }}

# Custom options of the DB configuration
sonar_conf_db__custom: {}

# Default configuration
sonar_conf__default: "{{
  sonar_conf_db__default.update(sonar_conf_db__custom) }}{{
  sonar_conf_db__default }}"

# Custom configuration
sonar_conf__custom: {}

# Final configuration
sonar_conf: "{{
  sonar_conf__default.update(sonar_conf__custom) }}{{
  sonar_conf__default }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`postgresql`](http://github.com/jtyr/ansible-postgresql) (optional)


License
-------

MIT


Author
------

Jiri Tyr
