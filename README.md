# Ansible role [passenger](https://galaxy.ansible.com/ui/standalone/roles/buluma/passenger/documentation)

Install and Configure Passenger with Nginx.

|GitHub|Version|Issues|Pull Requests|Downloads|
|------|-------|------|-------------|---------|
|[![github](https://github.com/buluma/ansible-role-passenger/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-passenger/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-passenger.svg)](https://github.com/buluma/ansible-role-passenger/releases/)|[![Issues](https://img.shields.io/github/issues/buluma/ansible-role-passenger.svg)](https://github.com/buluma/ansible-role-passenger/issues/)|[![PullRequests](https://img.shields.io/github/issues-pr-closed-raw/buluma/ansible-role-passenger.svg)](https://github.com/buluma/ansible-role-passenger/pulls/)|[![Ansible Role](https://img.shields.io/ansible/role/d/buluma/passenger)](https://galaxy.ansible.com/ui/standalone/roles/buluma/passenger/documentation)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/buluma/ansible-role-passenger/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true

  vars:
    passenger_server_name: local.example.test
    passenger_app_env: development
    passenger_app_root: /opt/demo/public
    nodejs_install_npm_user: root

  pre_tasks:
    - name: Update apt cache.
      apt: update_cache=true cache_valid_time=600
      when: ansible_os_family == 'Debian'

    - name: Ensure build dependencies are installed (Debian).
      apt:
        name:
          - build-essential
          - apt-transport-https
          - curl
        state: present
      when: ansible_os_family == 'Debian'

  roles:
    - role: buluma.bootstrap
    - role: buluma.python_pip
    - role: buluma.ruby_gems
    - role: buluma.nodejs
    # - role: buluma.passenger

  tasks:
    - name: Install rails dependencies.
      apt:
        name:
          - zlib1g-dev
          - libsqlite3-dev
          - ruby-dev
          - tzdata
        state: present

    # Install Ruby Version 2.3.1
    - name: Install ruby 2.3.1
      ansible.builtin.command: rbenv install {{ rbenv_ruby_version }}
      args:
        creates: '{{ rbenv_root }}/versions/{{ rbenv_ruby_version }}/bin/ruby'
      vars:
        rbenv_root: /usr/local/rbenv
        rbenv_ruby_version: 2.3.1
      environment:
        CONFIGURE_OPTS: '--disable-install-doc'
        RBENV_ROOT: '{{ rbenv_root }}'
        PATH: '{{ rbenv_root }}/bin:{{ rbenv_root }}/shims:{{ rbenv_plugins }}/ruby-build/bin:{{ ansible_env.PATH }}'

    - name: Check if Rails is already present.
      command: which rails
      register: rails_result
      changed_when: false
      failed_when: false

    - name: Install rails.
      command: gem install rails --no-document
      when: rails_result.rc != 0

    - name: Check if Rails app exists.
      stat: "path={{ passenger_app_root }}"
      register: rails_app

    - name: Create Rails demo app if it doesn't yet exist.
      command: "rails new demo --skip-git --skip-javascript chdir=/opt"
      when: not rails_app.stat.exists

    - name: Set correct permissions on demo app.
      file:  # noqa 208
        path: /opt/demo
        owner: "{{ nginx_user }}"
        group: "{{ nginx_user }}"
        state: directory
        recurse: true
      notify: restart nginx

  post_tasks:
    - name: Ensure Rails demo app is accessible.
      uri:
        url: "http://127.0.0.1/"
        status_code: 200
      register: result
      until: result.status == 200
      retries: 60
      delay: 1
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/buluma/ansible-role-passenger/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: yes
  gather_facts: no

  roles:
    - role: buluma.bootstrap
    - role: buluma.python_pip
```

Also see a [full explanation and example](https://buluma.github.io/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/buluma/ansible-role-passenger/blob/master/defaults/main.yml):

```yaml
---
# defaults file for passenger
passenger_server_name: www.example.com
passenger_app_root: /opt/example/public
passenger_app_env: production

passenger_root: /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini
passenger_ruby: /usr/bin/ruby

nginx_worker_processes: "{{ ansible_processor_vcpus | default(ansible_processor_count) }}"
nginx_worker_connections: "768"
nginx_keepalive_timeout: "65"
nginx_remove_default_vhost: true
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/buluma/ansible-role-passenger/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | Version |
|-------------|--------|--------|
|[buluma.bootstrap](https://galaxy.ansible.com/buluma/bootstrap)|[![Ansible Molecule](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-bootstrap.svg)](https://github.com/shadowwalker/ansible-role-bootstrap)|
|[buluma.python_pip](https://galaxy.ansible.com/buluma/python_pip)|[![Ansible Molecule](https://github.com/buluma/ansible-role-python_pip/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-python_pip/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-python_pip.svg)](https://github.com/shadowwalker/ansible-role-python_pip)|
|[buluma.ruby_gems](https://galaxy.ansible.com/buluma/ruby_gems)|[![Ansible Molecule](https://github.com/buluma/ansible-role-ruby_gems/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-ruby_gems/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-ruby_gems.svg)](https://github.com/shadowwalker/ansible-role-ruby_gems)|
|[buluma.nodejs](https://galaxy.ansible.com/buluma/nodejs)|[![Ansible Molecule](https://github.com/buluma/ansible-role-nodejs/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-nodejs/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-nodejs.svg)](https://github.com/shadowwalker/ansible-role-nodejs)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://buluma.github.io/) for further information.

Here is an overview of related roles:

![dependencies](https://raw.githubusercontent.com/buluma/ansible-role-passenger/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/buluma):

|container|tags|
|---------|----|
|[Fedora](https://hub.docker.com/repository/docker/buluma/fedora/general)|all|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/buluma/ansible-role-passenger/issues)

## [Changelog](#changelog)

[Role History](https://github.com/buluma/ansible-role-passenger/blob/master/CHANGELOG.md)

## [License](#license)

[Apache-2.0](https://github.com/buluma/ansible-role-passenger/blob/master/LICENSE)

## [Author Information](#author-information)

[Shadow Walker](https://buluma.github.io/)

