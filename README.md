stratis
=========

<img src="https://docs.ansible.com/ansible-tower/3.2.4/html_ja/installandreference/_static/images/logo_invert.png" width="10%" height="10%" alt="Ansible logo" align="right"/>
<a href="https://travis-ci.org/robertdebock/ansible-role-stratis"><img src="https://travis-ci.org/robertdebock/ansible-role-stratis.svg?branch=master" alt="Build status" align="left"/></a>

Install stratis and carves pools and filesystems.

<img src="https://img.shields.io/ansible/role/d/40309"/>
<img src="https://img.shields.io/ansible/quality/40309"/>

Example Playbook
----------------

This example is taken from `molecule/resources/playbook.yml`:
```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - role: robertdebock.stratis
      stratis_pools:
        - name: my_pool
          devices:
            - /dev/vdb
            - /dev/vdc
      stratis_filesystems:
        - name: my_filesystem
          pool: my_pool
      stratis_mounts:
        - mountpoint: /mnt/my_mountpoint
          device: /stratis/my_pool/my_filesystem
```

The machine you are running this on, may need to be prepared.
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

This role uses the following modules:
```yaml
---
- command
- file
- mount
- package
- service
```

Context
-------

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://robertdebock.nl/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/robertdebock/drawings/artifacts/stratis.png "Dependency")


Compatibility
-------------

This role has been tested on these [container images](https://hub.docker.com/):

|container|tag|allow_failures|
|---------|---|--------------|
|centos|latest|no|
|fedora|latest|no|
|fedora|rawhide|yes|

This role has been tested on these Ansible versions:

- ansible~=2.7
- ansible~=2.8
- git+https://github.com/ansible/ansible.git@devel

The indicator '\~=' means [compatible with](https://www.python.org/dev/peps/pep-0440/#compatible-release). For example 'ansible\~=2.8' would pick the latest ansible-2.8, for example ansible-2.8.6.

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

[Unit tests](https://travis-ci.org/robertdebock/ansible-role-stratis) are done on every commit and periodically.

If you find issues, please register them in [GitHub](https://github.com/robertdebock/ansible-role-stratis/issues)

To test this role locally please use [Molecule](https://github.com/ansible/molecule):
```
pip install molecule
molecule test
```

To test on Amazon EC2, configure [~/.aws/credentials](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html) and set a region using `export AWS_REGION=eu-central-1` before running `molecule test --scenario-name ec2`.

There are many specific scenarios available, please have a look in the `molecule/` directory.

License
-------

Apache-2.0


Author Information
------------------

[Robert de Bock](https://robertdebock.nl/)
