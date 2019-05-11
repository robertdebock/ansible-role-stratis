stratis
=========

[![Build Status](https://travis-ci.org/robertdebock/ansible-role-stratis.svg?branch=master)](https://travis-ci.org/robertdebock/ansible-role-stratis)

Install stratis and carves pools and filesystems.

Example Playbook
----------------

This example is taken from `molecule/default/playbook.yml`:
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
            - /device1
            - /device2
      stratis_filesystems:
        - name: my_filesystem
          pool: my_pool
      stratis_mounts:
        - mountpoint: /mnt/my_mountpoint
          device: /stratis/my_pool/my_filesystem
```

The machine you are running this on, may need to be prepared. Tests have been done on machines prepared by this playbook:
```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: no

  vars:
    devices:
      - /device1
      - /device2

  roles:
    - robertdebock.bootstrap

  tasks:
    - name: create file
      command: dd if=/dev/zero of={{ item }} bs=512 count=1048576
      args:
        creates: "{{ item }}"
      with_items:
        - "{{ devices }}"

    - name: make blockdevice
      command: mknod {{ item }} b 45 0
      with_items:
        - "{{ devices }}"
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
- A recent version of Ansible. (Tests run on the last 3 release of Ansible.)

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

This role has been tested against the following distributions and Ansible version:

|distribution|ansible 2.6|ansible 2.7|ansible devel|
|------------|-----------|-----------|-------------|
|alpine-edge*|no|no|no*|
|alpine-latest|no|no|no*|
|archlinux|no|no|no*|
|centos-6|no|no|no*|
|centos-latest|no|no|no*|
|debian-latest|no|no|no*|
|debian-stable|no|no|no*|
|debian-unstable*|no|no|no*|
|fedora-latest|yes|yes|yes*|
|fedora-rawhide*|yes|yes|yes*|
|opensuse-leap|no|no|no*|
|ubuntu-devel*|no|no|no*|
|ubuntu-latest|no|no|no*|
|ubuntu-rolling|no|no|no*|

A single star means the build may fail, it's marked as an experimental build.

Testing
-------

[Unit tests](https://travis-ci.org/robertdebock/ansible-role-stratis) are done on every commit and periodically.

If you find issues, please register them in [GitHub](https://github.com/robertdebock/ansible-role-stratis/issues)

To test this role locally please use [Molecule](https://github.com/metacloud/molecule):
```
pip install molecule
molecule test
```

To test on Amazon EC2, configure [~/.aws/credentials](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html) and `export AWS_REGION=eu-central-1` before running `molecule test --scenario-name ec2`.

There are many specific scenarios available, please have a look in the `molecule/` directory.

Run the [ansible-galaxy](https://github.com/ansible/galaxy-lint-rules) and [my](https://github.com/robertdebock/ansible-lint-rules) lint rules if you want your change to be merges:

```shell
git clone https://github.com/ansible/ansible-lint.git /tmp/ansible-lint
ansible-lint -r /tmp/ansible-lint/lib/ansiblelint/rules .

git clone https://github.com/robertdebock/ansible-lint /tmp/my-ansible-lint
ansible-lint -r /tmp/my-ansible-lint/rules .
```

License
-------

Apache-2.0


Author Information
------------------

[Robert de Bock](https://robertdebock.nl/) <robert@meinit.nl>
