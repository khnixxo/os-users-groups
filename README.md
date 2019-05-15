khnixxo.os_users_groups
=========

Managing Operating System Users and Groups in a flexible way across multiple host having different needs.

This role tries to achieves the following goals:
- Manage different users and groups on multiple hosts
- User and groups properties must be shared across all host. 
  - Don't repeat yourself
  - A change made on a property must take effect everywhere unless the property was overridden
  - Shared properties does mean that same user and groups are installed everywhere
- Must be able to override a specific property on a host without having to redefine the whole user/group
  - User account can have different groups in staging vs production
- The host managed user list must be clear and simple.

Requirements
------------

No requirements

Role Variables
--------------

**Groups Definition**

We define all the possible groups in this list. Theses groups won't be installed automatically.
Its a list of groups that you can choose from later.
A group must be in this list to be able to manage it later.
Its basically a list of Ansible `Group` module item
   
    khnixxo_os_groups_definition: []


**Users Definition**

We define all the possible users in this list. Theses users won't be installed automatically.
Its a list of users that you can choose from later.
Its basically a list of Ansible `user` module plus the authorized_keys property that is a list of Ansible `authorized_key` module item
   
    khnixxo_os_users_definition: []
    
    
**Groups Selection**

This is where we select the actual users we want to manage on a host or a group of host.
It is possible to override a definition property here to apply it on a specific host.
The overriding value will replace the one from definition.
Any property that is not overridden will be taken from the definition.

    khnixxo_os_groups_selection: []
    
    
**Users Selection**

This is where we select the actual users we want to manage on a host or a group of host.
It is possible to override a definition property here to apply it on a specific host.
The overriding value will replace the one from definition.
Any property that is not overridden will be taken from the definition.

    khnixxo_os_users_selection: []
    
Dependencies
------------

No Dependencies

Example Playbook
----------------



Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
# File: example-definition-vars.yml

khnixxo_os_groups_definition:
  - name: groupA
    state: present
  - name: groupB
    state: present
  - name: groupC
    state: present

khnixxo_os_users_definition:
  - name: ansible
    comment: User for configuration management
    groups:
      - sudo
    append: No
    shell: /bin/bash
    state: present
    authorized_keys:
      - name: id_rsa_ansible_control_machine
        comment: Used by the Control Machine to connect using ssh
        key: "ssh-rsa AAAAB3NzaC1MtDJ5W[...]95T+8sWqRTpR8D3+Cek+rzp4uPz66w=="
        state: present

  - name: user1
    groups:
      - sudo
      - groupA
      - groupB
      - groupC
    state: present

  - name: user2
    groups:
      - sudo
      - groupB
      - groupC
    state: present

  - name: user3
    groups:
      - groupA
    state: present

  - name: user4
    state: absent
```   


```yaml
# playbook.yml

---
- hosts: staging
  become: yes

  vars_files:
    - example-definition-vars.yml

  vars:
    khnixxo_os_groups_selection:
      - name: groupA
      - name: groupB

    khnixxo_os_users_selection:
      - name: ansible
      - name: user1
        groups:
          - sudo
          - groupA
          - groupB
      - name: user2
        groups:
          - sudo
          - groupB
      - name: user3
      - name: user4

  roles:
    - khnixxo.os_users_groups

- hosts: production
  become: yes

  vars_files:
    - example-definition-vars.yml

  vars:
    khnixxo_os_groups_selection:
      - name: groupA
      - name: groupC

    khnixxo_os_users_selection:
      - name: ansible
      - name: user1
        groups:
          - groupA
          - groupC
      - name: user2
        groups:
          - groupB
          - groupC
      - name: user3
        groups: []
      - name: user4

  roles:
    - khnixxo.os_users_groups

```
License
-------

MIT

Author Information
------------------

Kevin Hudon (khnixxo)
