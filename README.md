# Consul

Master: [![Build Status](https://travis-ci.org/sansible/consul.svg?branch=master)](https://travis-ci.org/sansible/consul)
Develop: [![Build Status](https://travis-ci.org/sansible/consul.svg?branch=develop)](https://travis-ci.org/sansible/consul)

* [Overview](#overview)
* [ansible.cfg](#ansible-cfg)
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
* [Settings](#settings)
* [Examples](#examples)

This roles installs Consul.

For more information on Consul please visit [consul.io](https://www.consul.io/).




## Overview

This role can install Consul in either agent or server mode. An AWS
autodiscovery script is included so Consul servers can be auto
detected using AWS tags.

If you want to use Consul as a nameserver there is also an option
to install Dnsmasq and set Consul as the main nameserver.

By default leave_on_terminate is set to true, this is so machines that
get recycled by an AWS Autoscaling group will leave the cluster properly.




## ansible.cfg

This role is designed to work with merge "hash_behaviour". Make sure your
ansible.cfg contains these settings

```INI
[defaults]
hash_behaviour = merge
```




## Installation and Dependencies

To install run `ansible-galaxy install sansible.consul` or add this to your
`roles.yml`:

```YAML
- name: sansible.consul
  version: v1.0
```

and run `ansible-galaxy install -p ./roles -r roles.yml`




## Tags

This role uses two tags: **build** and **configure**

* `build` - Installs Consul and all it's dependencies.
* `configure` - Configure and ensures that the Consul service is running.




## Settings

```YAML
---

consul:
  # Used to lookup consul server machines in AWS
  # Both agents and servers need to be able to lookup
  aws_ips_autodiscover:
    enabled: false
    lookup_filter: "Name=tag:Environment,Values={ENV} Name=tag:Role,Values=consul_server"
  # Useful when you have multiple private interfaces
  bind_interface: eth0
  checksum: "abdf0e1856292468e2c9971420d73b805e93888e006c76324ae39416edcf0627"
  directories:
    config_dir: "/etc/consul.d"
    data_dir: "/var/lib/consul"
    install_dir: "/usr/share/consul"
    log_dir: "/var/log/consul"
  # These get written to json config file
  settings:
    bootstrap: "false"
    bootstrap_expect: "1"
    datacenter: "aws-eu-west-1"
    enable_debug: "false"
    leave_on_terminate: "true"
    log_level: "warn"
    node_name: "{{ ansible_fqdn }}"
    rejoin_after_leave: "true"
    ports: {}
    server: "false"
    start_join: []
  startopts: ""
  # If set to true will install dnsmasq and set as resolver
  use_as_nameserver: no
  version: "0.6.4"
```




## Examples

To install in server mode:

```YAML
- name: Install and configure Consul Server
  hosts: "somehost"

  roles:
    - role: sansible.consul
      consul:
        settings:
          bootstrap_expect: 3
          server: "true"
          start_join: [ "some-host" ]
```

To install in agent mode:

```YAML
- name: Install and configure Consul Agent
  hosts: "somehost"

  roles:
    - role: sansible.consul
      consul:
        settings:
          server: "false"
          start_join: [ "some-host" ]
```

To use AWS autodiscovery:

```YAML
- name: Install and configure Consul Server
  hosts: "somehost"

  roles:
    - role: sansible.consul
      consul:
        aws_ips_autodiscover:
          enabled: true
          lookup_filter: "Name=tag:Environment,Values=dev Name=tag:Role,Values=consul_server"
        settings:
          bootstrap_expect: 3
          server: "true"
```
