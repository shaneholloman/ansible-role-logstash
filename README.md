# Ansible Role: Logstash

[![CI](https://github.com/shaneholloman-org/ansible-role-logstash/actions/workflows/ci.yml/badge.svg)](https://github.com/shaneholloman-org/ansible-role-logstash/actions/workflows/ci.yml)

An Ansible Role that installs Logstash on RedHat/CentOS Debian/Ubuntu.

Note that this role installs a syslog grok pattern by default; if you want to add more filters, please add them inside the `/etc/logstash/conf.d/` directory. As an example, you could create a file named `13-myapp.conf` with the appropriate grok filter and restart logstash to start using it. Test your grok regex using the [Grok Debugger](http://grokdebug.herokuapp.com/).

## Requirements

Though other methods are possible, this role is made to work with Elasticsearch as a backend for storing log messages.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    logstash_version: '7.x'

The major version of Logstash to install.

    logstash_package: logstash

The specific package to be installed. You can specify a version of the package using the correct syntax for your platform and package manager by changing the package name.

    logstash_listen_port_beats: 5044

The port over which Logstash will listen for beats.

    logstash_elasticsearch_hosts:
      - http://localhost:9200

The hosts where Logstash should ship logs to Elasticsearch.

    logstash_dir: /usr/share/logstash

The directory inside which Logstash is installed.

    logstash_ssl_dir: /etc/pki/logstash
    logstash_ssl_certificate_file: logstash-forwarder-example.crt
    logstash_ssl_key_file: logstash-forwarder-example.key

Local paths to the SSL certificate and key files, which will be copied into the `logstash_ssl_dir`.

See [Generating a self-signed certificate](#generating-a-self-signed-certificate) for information about generating and using self-signed certs with Logstash and Filebeat.

    logstash_local_syslog_path: /var/log/syslog
    logstash_monitor_local_syslog: true

Whether configuration for local syslog file (defined as `logstash_local_syslog_path`) should be added to logstash. Set this to `false` if you are monitoring the local syslog differently, or if you don't care about the local syslog file. Other local logs can be added by your own configuration files placed inside `/etc/logstash/conf.d`.

    logstash_enabled_on_boot: true

Set this to `false` if you don't want logstash to run on system startup.

    logstash_install_plugins:
      - logstash-input-beats
      - logstash-filter-multiline

A list of Logstash plugins that should be installed.

    logstash_setup_default_config: true

Set this to `false` if you don't want to add the default config files shipped with this role (inside the `files/filters` directory). You can add your own configuration files inside `/etc/logstash/conf.d`.

## Generating a Self-signed certificate

For utmost security, you should use your own valid certificate and keyfile, and update the `logstash_ssl_*` variables in your playbook to use your certificate.

To generate a self-signed certificate/key pair, you can use use the command:

    $ openssl req -x509 -batch -nodes -days 3650 -newkey rsa:2048 -keyout logstash.key -out logstash.crt -subj '/CN=example.com'

Note that Filebeat and Logstash may not work correctly with self-signed certificates unless you also have the full chain of trust (including the Certificate Authority for your self-signed cert) added on your server. See: https://github.com/elastic/logstash/issues/4926#issuecomment-203936891

Newer versions of Filebeat and Logstash also require a pkcs8-formatted private key, which can be generated by converting the key generated earlier, e.g.:

    openssl pkcs8 -in logstash.key -topk8 -nocrypt -out logstash.p8

## Other Notes

If you are seeing high CPU usage from one of the `logstash` processes, and you're using Logstash along with another application running on port 80 on a platform like Ubuntu with upstart, the `logstash-web` process may be stuck in a loop trying to start on port 80, failing, and trying to start again, due to the `restart` flag being present in `/etc/init/logstash-web.conf`. To avoid this problem, either change that line to add a `limit` to the respawn statement, or set the `logstash-web` service to `enabled=no` in your playbook, e.g.:

    - name: Ensure logstash-web process is stopped and disabled.
      service: name=logstash-web state=stopped enabled=no

## Example Playbook

    - hosts: search
    
      pre_tasks:
        - name: Use Java 8 on Debian/Ubuntu.
          set_fact:
            java_packages:
              - openjdk-8-jdk
          when: ansible_os_family == 'Debian'
    
      roles:
        - shaneholloman.java
        - shaneholloman.elasticsearch
        - shaneholloman.logstash

## License

MIT / BSD

## Author Information

This role was created in 2023

