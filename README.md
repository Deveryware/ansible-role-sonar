# Ansible Role: SonarQube

An Ansible Role that installs [SonarQube](http://www.sonarqube.org/) on RedHat/CentOS and Debian/Ubuntu Linux servers.

## Requirements

Requires the `unzip` utility to be installed on the server. Also, different SonarQube versions require different minimum versions of Java:

  - SonarQube 5.0-5.5 requires Java 7+
  - SonarQube 5.6-7.8 requires Java 8+
  - SonarQube 7.9+ requires Java 11+
  - Sonarqube 9.9+ requires Java 17+

Same thing for databases versions:

  - SonarQube 6.7-7.2 requires MySQL 5.6+ or PostgreSQL 8+
  - SonarQube 7.3-7.8 requires MySQL 5.6+ or PostgreSQL 9.3+
  - SonarQube 7.9 dropped support for MySQL entirely

## Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `sonar_base_api_url` | URL to SonarQube API | `http://localhost:{{ sonar_configuration.sonar.web.port }}{{ sonar_configuration.sonar.web.context \| default('') }}/api` |
| `sonar_community_plugins` | List of community plugins to install. See below | `[]` |
| `sonar_configuration` | See below | ... |
| `sonar_download_url` | Source for SonarQube releases | `https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-{{ sonar_version }}.zip` |
| `sonar_download_validate_certs` | Validate certificates for `sonar_download_url` | `true` |
| `sonar_group` | UNIX sonar group | `sonar` |
| `sonar_install_method` | See below | `copy` |
| `sonar_plugins` | List of official plugins to intsall. See below | `[]` |
| `sonar_tmp_dest` | Absolute path of where temporary file is downloaded to. | [`tmp_dest`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html)'s default |
| `sonar_token` | Sonar security token, used to install plugins | `""` |
| `sonar_user` | UNIX sonar user | `sonar` |
| `sonar_version_directory` | Name of the installation directory | `sonarqube-{{ sonar_version }}` |
| `sonar_version` | SonarQube version to install, accepts a numerical value corresponding to a known release or the special value `latest`. See below | `8.9` |
| `workspace` | Temporary storage area for downloaded files | `/tmp` |

### sonar_install_method

This variable dictate how the files from a new release are installed in the destination folder `/usr/local/sonar`.

First they are extracted to `/usr/local/{{ sonar_version_directory }}`.

Then, if `sonar_install_method=copy` the source folder is recursively copied into the destination folder using the module `copy`.

The other possibility is `sonar_install_method=link` where the source folder is symlinked to the destination folder, preserving any previously installed version.

### sonar_configuration

Yaml representation of SonarQube configuration.

Example:

```yaml
sonar_configuration:
  sonar:
    web:
      host: 0.0.0.0
      port: 9000
    jdbc:
      url: "jdbc:mysql://localhost:3306/sonar"
```

Translates to:

    sonar.web.host=0.0.0.0
    sonar.web.port=9000
    sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar

This way it is possible to configure LDAP, HTTP proxies, MySQL, PostgreSQL, and more.

Example:

```yaml
sonar_configuration:
  sonar:
    web:
      host: 0.0.0.0
      port: 9000
      context: /sonar
    jdbc:
      username: sonar
      password: "{{ vaulted_password }}"
      url: "jdbc:mysql://localhost:3306/sonar"
    ce:
      javaAdditionalOpts: -javaagent:...
    security:
      realm: LDAP
  ldap:
    url: "{{ ldap_url }}"
    bindDn: "{{ bind_dn }}"
    bindPassword: "{{ vaulted_bind_password }}"
    user:
      baseDn: ou=Users,dc=example,dc=org
      realNameAttribute: cn
      emailAttribute: mail
    group:
      request: (&(objectClass=posixGroup)(memberUid={uid}))
  http:
    proxyHost: "{{ proxy_host }}"
    proxyPort: "{{ proxy_port }}"
  https:
    proxyHost: "{{ proxy_host }}"
    proxyPort: "{{ proxy_port }}"
```

### sonar_plugins

This variable is a list of dict describing a particular plugin:

| Key | Description | Mandatory? |
|-----|-------------|------------|
| name | Plugin name | yes |
| state | Either `absent` or `present` | no |

It is mandatory to define a security token for an administrator account in the variable `sonar_token` for this functionality to work.

Example:

```yaml
sonar_plugins:
- name: Git
- name: SonarPython
- name: Svn
  state: absent
- name: Cvs
  state: absent
```

### sonar_community_plugins

This variable is a list of dict describing a particular plugin from the community:

| Key | Description | Mandatory? |
|-----|-------------|------------|
| name | Plugin name | yes |
| url | An URL pointing to jar | yes |
| state | Either `absent` or `present` | no |

Example:

```yaml
sonar_community_plugins:
- name: backelite-sonar-swift
  url: https://.../backelite-sonar-swift-plugin-0.4.5.jar
- name: sonar-bad
  url: https://.../sonar-bad-0.4.2.jar
  state: absent
```

Note that the key `url` is mandatory, even when `state=absent`.

### sonar_version

This variable contains the SonarQube version to install or upgrade.
It must contains the full version number as displayed in the download link, ie: `8.1.0.31237` for versions superior to 8.0.

For automatic upgrades the special value `latest` is accepted but an administrator token must configured, just like `sonar_plugins`.
In this case the role will select the next available version using SonarQube API.

## Dependencies

A role dedicated to the configuration of either mysql or postgresql, such as:

- [geerlingguy/ansible-role-mysql](https://github.com/geerlingguy/ansible-role-mysql)
- [geerlingguy/ansible-role-postgresql](https://github.com/geerlingguy/ansible-role-postgresql)
- [ANXS/postgresql](https://github.com/ANXS/postgresql)

A role dedicated to the configuration of java, such as:

- [geerlingguy/ansible-role-java](https://github.com/geerlingguy/ansible-role-java)

## Example Playbooks

```yaml
- hosts: all
  roles:
    - ansible-role-mysql
    - ansible-role-java
    - ansible-role-sonar
      sonar_version: 8.9.2.46101
```

or

```yaml
- hosts: all
  roles:
    - ansible-role-postgresql
    - ansible-role-java
    - ansible-role-sonar
```

Using the defaults, you can view the SonarQube home at `http://localhost:9000/` (default System administrator credentials are `admin`/`admin`).

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
It was forked in 2019 by [Tristan Le Guern](https://www.bouledef.eu/~tleguern) and [Olivier Locard](https://github.com/olo-dw) on behalf of [Deveryware](https://deveryware.com).
