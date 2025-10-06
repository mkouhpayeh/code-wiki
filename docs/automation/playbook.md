``` shell title="ansible.cfg"
[defaults]

# disable SSH key host checking
host_key_checking = False

# smart - gather by default, but don't regather if already gathered
gathering = smart
```

``` shell title="create a new file with a name for example inventory - put in the same directory of playbooks"
[groupName]
hostName ansible_host=10.0.0.1

[groupName:vars]
ansible_user=ansible
ansible_ssh_pass=***
ansible_become_pass=*** //Not best practice
```

``` yaml title="variables.yml"
---
- name: A simple playbook to add users to the host
  hosts: LL-Test
  gather-facts: false
  vars_files:
    - files/users.yml
  vars:
    # new_user = test1
  tasks:
    - name: Add user to the host
      ansible.builtin.user:
        # name: {{ new_user }}
        # use the variable stored in inventory : LL-Test ansible_host=10.0.0.1 new_user=test1
        name: {{ hostvars[inventory_hostname].new_user }}
        state: present
      become: true
```
> ansible-playbook -i inventory -e "new_user=test1" variables.yml

``` yaml title="loop.yml"
---
- name: A simple playbook to add users to the host
  hosts: LL-Test
  gather-facts: false
  vars:
    new_user
      - test1
      - test2
      - test3
  tasks:
    - name: Add user to the host
      ansible.builtin.user:
        name: {{ item }}
        state: present
      become: true
      loop: {{ new_user }}
        # - test1
        # - test2
        # - test3
```
