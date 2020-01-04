stratis
=========

<img src="https://docs.ansible.com/ansible-tower/3.2.4/html_ja/installandreference/_static/images/logo_invert.png" width="10%" height="10%" alt="Ansible logo" align="right"/>
<a href="https://travis-ci.org/robertdebock/ansible-role-stratis"> <img src="https://travis-ci.org/robertdebock/ansible-role-stratis.svg?branch=master" alt="Build status"/></a> <img src="https://img.shields.io/ansible/role/d/40309"/> <img src="https://img.shields.io/ansible/quality/40309"/>

<a href="https://github.com/robertdebock/ansible-role-stratis/actions"><img src="https://github.com/robertdebock/ansible-role-stratis/workflows/GitHub%20Action/badge.svg"/></a>

Install stratis and carves pools and filesystems.

Example Playbook
----------------

This example is taken from `molecule/resources/playbook.yml` and is tested on each push, pull request and release.
```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  # vars:
  #   stratis_pools:
  #     - name: my_pool
  #       devices:
  #         - /dev/vdb
  #         - /dev/vdc
  #   stratis_filesystems:
  #     - name: my_filesystem
  #       pool: my_pool
  #   stratis_mounts:
  #     - mountpoint: /mnt/my_mountpoint
  #       device: /stratis/my_pool/my_filesystem

  roles:
    - role: robertdebock.stratis
```

The machine you are running this on, may need to be prepared, I use this playbook to ensure everything is in place to let the role work.
```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: no

  vars:
    devices:
      - name: vdb
        major: 252
        minor: 2
      - name: vdc
        major: 252
        minor: 3

  roles:
    - robertdebock.bootstrap

  tasks:
    - name: create storage file
      command: dd if=/dev/zero of=/{{ item.name }} bs=1M count=1K
      args:
        creates: "/{{ item.name }}"
      with_items:
        - "{{ devices }}"
      notify:
        - create loopback device
        - loopback device to storage file
      loop_control:
        label: "/{{ item.name }}"

  handlers:
    - name: create loopback device
      command: mknod /dev/{{ item.name }} b {{ item.major }} {{ item.minor }}
      with_items:
        - "{{ devices }}"
      loop_control:
        label: "/dev/{{ item.name }}"

    - name: loopback device to storage file
      command: losetup /dev/{{ item.name }} /{{ item.name }}
      with_items:
        - "{{ devices }}"
      failed_when: no
      register: stratis_loopback_device_to_storage_file
      until: stratis_loopback_device_to_storage_file is succeeded
      retries: 3
      loop_control:
        label: "/dev/{{ item.name }} to /{{ item.name }}"
```

After running this role, this playbook runs to verify that everything works, this may be a good example how you can use this role.
```yaml
---
- name: Verify
  hosts: all
  become: yes
  gather_facts: yes

  tasks:
    - name: stratis version
      command: stratis --version
```

Also see a [full explanation and example](https://robertdebock.nl/how-to-use-these-roles.html) on how to use these roles.

Role Variables
--------------

These variables are set in `defaults/main.yml`:
```yaml
---
# defaults file for stratis
```

Requirements
------------

- Access to a repository containing packages, likely on the internet.
- A recent version of Ansible. (Tests run on the current, previous and next release of Ansible.)

The following roles can be installed to ensure all requirements are met, using `ansible-galaxy install -r requirements.yml`:

```yaml
---
- robertdebock.bootstrap

```

Context
-------

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://robertdebock.nl/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/robertdebock/drawings/artifacts/stratis.png "Dependency")


Compatibility
-------------

This role has been tested on these [container images](https://hub.docker.com/):

|container|tags|
|---------|----|
|el|8|
|fedora|all|

The minimum version of Ansible required is 2.8 but tests have been done to:

- The previous version, on version lower.
- The current version.
- The development version.

Exceptions
----------

Some variarations of the build matrix do not work. These are the variations and reasons why the build won't work:

| variation                 | reason                 |
|---------------------------|------------------------|
| Alpine | Stratis is not available. |
| CentOS-7 | Stratis is not available. |
| Debian | Stratis is not available. |
| openSUSE Leap | Stratis is not available. |
| Ubuntu | Stratis is not available. |


Testing
-------

[Unit tests](https://travis-ci.org/robertdebock/ansible-role-stratis) are done on every commit, pull request, release and periodically.

If you find issues, please register them in [GitHub](https://github.com/robertdebock/ansible-role-stratis/issues)

Testing is done using [Tox](https://tox.readthedocs.io/en/latest/) and [Molecule](https://github.com/ansible/molecule):

[Tox](https://tox.readthedocs.io/en/latest/) tests multiple ansible versions.
[Molecule](https://github.com/ansible/molecule) tests multiple distributions.

To test using the defaults (any installed ansible version, namespace: `robertdebock`, image: `fedora`, tag: `latest`):

```
molecule test

# Or select a specific image:
image=ubuntu molecule test
# Or select a specific image and a specific tag:
image="debian" tag="stable" tox
```

Or you can test multiple versions of Ansible, and select images:
Tox allows multiple versions of Ansible to be tested. To run the default (namespace: `robertdebock`, image: `fedora`, tag: `latest`) tests:

```
tox

# To run CentOS (namespace: `robertdebock`, tag: `latest`)
image="centos" tox
# Or customize more:
image="debian" tag="stable" tox
```

License
-------

Apache-2.0


Author Information
------------------

[Robert de Bock](https://robertdebock.nl/)
