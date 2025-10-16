# Ansible Playbook

An Ansible playbook is a YAML file that defines a set of tasks to automate configuration and management of one or more hosts.

## Linux Hosts

```shell title="ansible.cfg"
[defaults]
# disable SSH key host checking
host_key_checking = False

# smart - gather by default, but don't regather if already gathered
gathering = smart
```

```ini title="inventory (put in the same directory as playbooks)"
[rocky9]  # group name
LL-rocky9-01 ansible_host=10.0.50.43 new_user=test1

[rocky9:vars]
ansible_user=ansible
ansible_ssh_pass=gregsowell
ansible_become_pass=gregsowell

[windows]
windows-01 ansible_host=10.0.50.31

[windows:vars]
ansible_connection=winrm
ansible_winrm_scheme=http
ansible_port=5985
ansible_winrm_transport=ntlm
ansible_user=administrator
ansible_become_method=runas
ansible_become_user=administrator

[nexus]
sw1 ansible_host=10.0.50.27

[nexus:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=cisco.nxos.nxos
ansible_user=admin
ansible_ssh_pass=lab
ansible_become_pass=lab
```

```yaml title="variables.yml"
---
- name: A simple playbook to add users to the host
  hosts: LL-Test
  gather_facts: false
  vars_files:
    - files/users.yml
  vars:
    # new_user = test1
  tasks:
    - name: Add user to the host
      ansible.builtin.user:
        # use variable stored in inventory: LL-Test new_user=test1
        name: "{{ hostvars[inventory_hostname].new_user }}"
        state: present
      become: true
```
Run:
```bash
ansible-playbook -i inventory -e "new_user=test1" variables.yml
```

```yaml title="loop.yml"
---
- name: A simple playbook to add users to the host
  hosts: LL-Test
  gather_facts: false
  vars:
    new_user:
      - test1
      - test2
      - test3
  tasks:
    - name: Add user to the host
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      become: true
      loop: "{{ new_user }}"
```

```yaml title="conditions.yml"
---
- name: A simple playbook to add users to the host
  hosts: LL-Test
  gather_facts: false
  vars_files:
    - files/users.yml
  vars:
    ignore_users:
      - root
      - test1
    new_user:
      - test1
      - test2
      - test3
  tasks:
    - name: Add user to the host (skip ignored)
      when: item not in (ignore_users | default([]))
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      become: true
      loop: "{{ new_user }}"
```

```yaml title="blocks.yml"
---
- name: Add nginx webserver to hosts
  hosts: LL-Test
  become: true
  tasks:
    - name: Block for Rocky hosts
      when: "'Rocky' in ansible_facts['distribution'] | default('')"
      block:
        - name: Install nginx webserver
          ansible.builtin.dnf:
            name: nginx
            state: present
        - name: Start and enable nginx service
          ansible.builtin.systemd:
            name: nginx
            enabled: yes
            state: started

    - name: Block for Ubuntu hosts
      when: "'Ubuntu' in ansible_facts['distribution'] | default('')"
      block:
        - name: Install nginx webserver
          ansible.builtin.apt:
            name: nginx
            state: present
        - name: Start and enable nginx service
          ansible.builtin.systemd:
            name: nginx
            enabled: yes
            state: started
```

**Block sections**

- **always:** runs after the block's tasks complete. It runs even if some tasks failed.
- **rescue:** runs only if a failure occurred inside the block.
- You cannot loop over a block. You can loop over tasks inside the block, but not the whole block.

```yaml title="templates.yml"
---
- name: Add nginx webserver and templates
  hosts: LL-Test
  become: true
  vars:
    owner_name: Test
    web_path: /home/webserver
  tasks:
    - name: Install nginx on Rocky
      when: "'Rocky' in ansible_facts['distribution'] | default('')"
      block:
        - name: Install nginx webserver
          ansible.builtin.dnf:
            name: nginx
            state: present
        - name: Start and enable nginx service
          ansible.builtin.systemd:
            name: nginx
            enabled: yes
            state: started

    - name: Create web directory
      ansible.builtin.file:
        path: "{{ web_path }}"
        state: directory
        owner: nginx
        group: nginx
        mode: '0755'

    - name: Add index.html to webserver directory
      ansible.builtin.template:
        src: templates/index.html.j2  # Save HTML with .j2 and use variables like {{ owner_name }}
        dest: "{{ web_path }}/index.html"
        owner: nginx
        group: nginx
        mode: '0644'

    - name: Add nginx config based on template
      ansible.builtin.template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: nginx
        group: nginx
        mode: '0644'
      register: nginx_config

    - name: Restart nginx service when config changed
      when: nginx_config.changed
      ansible.builtin.service:
        name: nginx
        state: restarted
```

**Template plugin**
```yaml
- name: Display template plugin
  ansible.builtin.debug:
    msg: "{{ lookup('ansible.builtin.template', './display-template.j2') }}"
```

**Handlers**

Handlers run once at the end of a play if notified by tasks that changed.

```yaml title="handlers.yml"
---
- name: Add nginx webserver to hosts (with handlers)
  hosts: LL-Test
  become: true
  vars:
    owner_name: Test
    web_path: /home/webserver
  tasks:
    - name: Create nginx config from template
      ansible.builtin.template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      register: nginx_config
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

**Tags**

Tags mark tasks so you can run subsets of a playbook:

```yaml title="tags.yml"
---
- name: Add nginx webserver to hosts (tags example)
  hosts: LL-Test
  become: true
  vars:
    owner_name: Test
    web_path: /home/webserver
  tasks:
    - name: Install nginx on Rocky
      when: "'Rocky' in ansible_facts['distribution'] | default('')"
      block:
        - name: Install nginx webserver
          ansible.builtin.dnf:
            name: nginx
            state: present
      tags: ['install']

    - name: Configure web directory
      ansible.builtin.file:
        path: "{{ web_path }}"
        state: directory
      tags: ['configure']
```
Run:
```bash
ansible-playbook -i inventory tags.yml --tags "install"
ansible-playbook -i inventory tags.yml --skip-tags "configure"
```

**Execution modes**

- **run:** normal execution (changes applied).
- **check:** dry-run — simulates changes. Note: some tasks do not support check mode.
  - To force specific tasks to run even in check mode, add `check_mode: false` at task level.

```yaml title="assert.yml"
---
- name: Check RAM on a Rocky host
  hosts: LL-Test
  tasks:
    - name: Display ansible_memtotal_mb
      ansible.builtin.debug:
        var: ansible_memtotal_mb

    - name: Assert if RAM is at least 10GB
      ansible.builtin.assert:
        that:
          - ansible_memtotal_mb > 10000
        fail_msg: "RAM is less than 10GB"
```

**Task-level control**
- `changed_when: false`
- `failed_when: false`

**Nested loop**

For complex nested loops prefer splitting logic into task includes or role tasks.

**Dynamic inventory**

A dynamic inventory script generates inventory data from external sources (ServiceNow, vCenter, AWS, etc.).

```yaml title="dynamic-inventory.yml"
---
- name: Dynamic Inventory
  hosts: all
  gather_facts: true
  tasks:
    - name: Print out the list of inventory hosts
      ansible.builtin.debug:
        var: groups['all']
      run_once: true
```

```python title="custom_inventory.py"
#!/usr/bin/env python
import json
# Read JSON payload from file
with open('files/json-payload.json', 'r') as file:
    json_payload = file.read()

inventory_data = json.loads(json_payload)
ansible_inventory = {
    '_meta': {'hostvars': {}},
    'all': {'hosts': [], 'vars': {}}
}
os_groups = {}
location_groups = {}

for host in inventory_data['hosts']:
    hostname = host['hostname']
    ip_address = host['ip_address']
    os_name = host['os']
    location = host['location']
    owner = host['owner']

    ansible_inventory['all']['hosts'].append(hostname)
    ansible_inventory['_meta']['hostvars'][hostname] = {
        'ansible_host': ip_address,
        'status': host['status'],
        'os': os_name,
        'location': location,
        'owner': owner
    }

    os_groups.setdefault(os_name, {'hosts': []})['hosts'].append(hostname)
    location_groups.setdefault(location, {'hosts': []})['hosts'].append(hostname)

ansible_inventory.update(os_groups)
ansible_inventory.update(location_groups)

print(json.dumps(ansible_inventory, indent=4))
```

```json title="json-payload.json"
{
  "hosts": [
    { "id": 1, "hostname": "host1.example.com", "ip_address": "192.168.1.101", "status": "active", "os": "Linux", "location": "dal-dc1", "owner": "Admin User" },
    { "id": 2, "hostname": "host2.example.com", "ip_address": "192.168.1.102", "status": "inactive", "os": "Windows", "location": "dal-dc1", "owner": "User 1" }
  ]
}
```
Run:
```bash
ansible-playbook -i files/custom_inventory.py dynamic-inventory.yml
```

**Sample play**

```yaml title="test.yml"
---
- name: Test play
  hosts: LL-rocky9-01
  vars:
    new_dirs:
      - /tmp/test10
      - /tmp/test11
      - /tmp/test12
      - /tmp/test13
  tasks:
    - name: Assert that the system has enough memory
      ansible.builtin.assert:
        that: ansible_memtotal_mb > 1000
        fail_msg: "The system does not have enough memory < 1GB"
        success_msg: "The system has enough memory > 1GB"

    - name: Create directories based on new_dirs variable
      ansible.builtin.file:
        path: "{{ item }}"
        state: directory
      loop: "{{ new_dirs }}"

    - name: Create a file with specific content idempotently if there is a 3 in the dir name
      ansible.builtin.copy:
        dest: "{{ item }}/file.txt"
        content: "This is the content of the file"
      when: "'3' in item"
      loop: "{{ new_dirs }}"

    - name: Collect the contents of the new_dirs
      ansible.builtin.shell: "ls -l {{ item }}"
      loop: "{{ new_dirs }}"
      register: ls_output
      changed_when: false

    - name: Display the contents of the new_dirs
      when: item.stdout | length > 8
      ansible.builtin.debug:
        var: item.stdout
      loop: "{{ ls_output.results }}"
```

## Roles

Roles abstract reusable parts of a playbook (like functions). Roles live in a `roles/` directory and follow a specific layout. Create a role with:

```bash
ansible-galaxy init nginx
```

Example role files:

```yaml title="roles/nginx/defaults/main.yml"
---
owner_name: Greg Sowell
web_path: /home/webserver
```

```yaml title="roles/nginx/handlers/main.yml"
---
- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

```yaml title="roles/nginx/tasks/main.yml"
---
- name: Block for rocky hosts
  when: "'Rocky' in ansible_facts['distribution'] | default('')"
  block:
    - name: Install nginx webserver
      ansible.builtin.dnf:
        name: nginx
        state: present
    - name: Start and enable nginx service
      ansible.builtin.systemd:
        name: nginx
        enabled: yes
        state: started

- name: Create /home/webserver directory
  ansible.builtin.file:
    path: "{{ web_path }}"
    state: directory
    owner: nginx
    group: nginx
    mode: '0755'

- name: Add nginx config based on template
  ansible.builtin.template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  register: nginx_config
  notify: Restart nginx
```

Run role via playbook:

```yaml title="nginx-role.yml"
- name: Add/Configure nginx webserver via Roles
  hosts: LL-rocky9-01
  become: true
  roles:
    - nginx
```
Run:
```bash
ansible-playbook -i inventory nginx-role.yml
```

**Role search path**
1. Local `roles/` directory (relative to playbook)
2. `ANSIBLE_ROLES_PATH` environment variable
3. `roles_path` in `ansible.cfg`
4. System-wide `/etc/ansible/roles`
5. Hidden Ansible Galaxy directory

**Acquire roles**
- Git clone
- Ansible Galaxy: `ansible-galaxy install <role_name>`
- Use a requirements file: `ansible-galaxy install -r requirements.yml`

## Secrets (Vault)

Encrypt secrets with Ansible Vault. Decrypt them at runtime with a vault password or vault ID.

```yaml title="files/secrets.yml"
---
super_secret: Greg
```

```bash
ansible-vault encrypt files/secrets.yml
ansible-vault decrypt files/secrets.yml
```

```yaml title="vault.yml"
---
- name: Vaulting example
  hosts: LL-rocky9-01
  gather_facts: false
  become: true
  vars_files:
    - files/secrets.yml  # Encrypted file
  tasks:
    - name: Display the super_secret variable
      ansible.builtin.debug:
        var: super_secret
```
Run:
```bash
ansible-playbook -i inventory vault.yml --ask-vault-pass
```

## Windows Hosts

- Connections via WinRM
- Patch management (WSUS, Intune, etc.)
- Configuration and app deployment (Active Directory tasks, security policies, PowerShell integration)

```yaml title="windows.yml"
---
- name: Perform windows updates
  hosts: windows-01
  gather_facts: false
  vars_prompt:
    - name: "ansible_ssh_pass"
      prompt: Windows password
      private: true
  tasks:
    - name: Update the windows server
      ansible.windows.win_updates:
        category_names:
          - CriticalUpdates
        state: searched
      register: update_output
```
Run:
```bash
ansible-playbook -i inventory windows.yml
```

## Network Hosts

```yaml title="network.yml"
---
- name: SNMP updates on a Cisco device
  hosts: sw1
  gather_facts: false
  vars:
    parse_me: |
      snmp-server user admin auth md5 0xe2bb3e2405ce601c5c3993262270ace5 priv 0xe2bb3e2405ce601c5c3993262270ace5 localizedkey engineID 128:0:0:9:3:0:80:86:135:240:25
      snmp-server community private group network-admin
      snmp-server community public group network-operator
  tasks:
    - name: Find current SNMP configuration
      cisco.nxos.nxos_command:
        commands: show run | inc snmp-ser
      register: snmp_output

    - name: Display current SNMP configuration
      ansible.builtin.debug:
        var: snmp_output.stdout_lines

    - name: Configure SNMP with the config module
      cisco.nxos.nxos_config:
        lines:
          - snmp-server community public
          - snmp-server community private
```

**Notes**
- You can replace parsed configs with resource modules where supported.
- Store desired CLI in files and apply via automation to keep changes auditable.

## APIs

Use the `uri` module to interact with HTTP APIs.

```yaml title="api.yml"
- name: Fetch current weather data
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Get weather data for College Station, Texas
      ansible.builtin.uri:
        url: "https://api.weather.gov/points/30.6280,-96.3344"
        method: GET
        validate_certs: yes
      register: weather_response

    - name: Get detailed forecast data
      ansible.builtin.uri:
        url: "{{ weather_response.json.properties.forecast }}"
        method: GET
        validate_certs: true
      register: forecast_response

    - name: Display detailed forecast data
      ansible.builtin.debug:
        var: forecast_response.json
```
