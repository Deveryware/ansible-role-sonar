# Ansible Role: SonarQube

[![Build Status](https://travis-ci.org/geerlingguy/ansible-role-sonar.svg?branch=master)](https://travis-ci.org/geerlingguy/ansible-role-sonar)

An Ansible Role that installs [SonarQube](http://www.sonarqube.org/) on RedHat/CentOS and Debian/Ubuntu Linux servers.

## Requirements

Requires the `unzip` utility to be installed on the server. Also, different SonarQube versions require different minimum versions of Java:

  - SonarQube 5.0-5.5 requires Java 1.7+
  - SonarQube 5.6+ requires Java 1.8+

Finally, recent versions of SonarQube also require MySQL 5.6 or later.

## Role Variables

Available variables are listed below, along with default values:

    workspace: /root

Directory where downloaded files will be temporarily stored.

    sonar_download_validate_certs: true

Controls whether to validate certificates when downloading SonarQube.

    sonar_download_url: http://dist.sonar.codehaus.org/sonarqube-4.5.4.zip
    sonar_version_directory: sonarqube-4.5.4

The URL from which SonarQube will be downloaded, and the resulting directory name (should match the download archive, without the archive extension).

    sonar_install_method: "move"

The way you want the install to be done. By default **move** is a rename of the versionned directory into _sonar_. You can set to **link** to create a symlink _sonar_ targeting the versionned directory. You can set to **copy** to copy the versionned directory into _sonar_ and remove the versionned directory (use case: dedicated filesystem).

    sonar_previous_version_backup: False

Wether to remove or not the previous version after an upgrade.

    sonar_user: sonar

Change Sonar default user.

    sonar_group: sonar

Change Sonar default group.

    sonar_configuration:
      sonar:
        host: 0.0.0.0
        port: 9000
      jdbc:
        url: "jdbc:mysql://localhost:3306/sonar"

A flexible yaml document representing SonarQube configuration.
It is possible to configure the ldap plugin, http proxy, and much more.

Example:

    sonar_configuration:
      sonar:
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
	...
      http:
        proxyHost: "{{ proxy_host }}"
        proxyPort: "{{ proxy_port }}"

## Dependencies

A role dedicated to the configuration of either mysql or postgresql, such as:

- [geerlingguy/ansible-role-mysql](https://github.com/geerlingguy/ansible-role-mysql)
- [ANXS/postgresql](https://github.com/ANXS/postgresql)

A role dedicated to the configuration of java, such as:

- [geerlingguy/ansible-role-java](https://github.com/geerlingguy/ansible-role-java)

## Example Playbook

    - hosts: all
      roles:
        - ansible-role-mysql
        - ansible-role-java
        - ansible-role-sonar

Using the defaults, you can view the SonarQube home at `http://localhost:9000/` (default System administrator credentials are `admin`/`admin`).

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
