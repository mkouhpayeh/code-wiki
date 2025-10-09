# Playbook
## Linux Hosts
``` shell title="ansible.cfg"
[defaults]
host_key_check = False
host_key_checking = False

# disable SSH key host checking
host_key_checking = False

# smart - gather by default, but don't regather if already gathered
gathering = smart
```

``` shell title="create a new file with a name for example inventory - put in the same directory of playbooks"
[rocky9] # It's the group name
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

``` yaml title="conditions.yml"
---
- name: A simple playbook to add users to the host
  hosts: LL-Test
  gather-facts: false
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
    - name: Add user to the host
      # when: item not in ignore_users and ignore_users is defined
      when: item not in ignore_users | default("alwaysworks") | default(omit)
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      become: true
```

``` yaml title="blocks.yml"
---
- name: Add nginx webserver to a Rocky host
  hosts: LL-Test
  become: true
  vars:
  tasks:
    - name: Block for Rocky hosts
      when: "'Rocky' in ansible_facts['distributions']"
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
      when: "'Ubuntu' in ansible_facts['distributions']"
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

- Block Section
  -  Always : will always be run after the block's task complete. Even if some tasks in the blocks fail and the playbook would normally be halted, the Always section still runs its tasks.
  -  Rescue : by default, none of the tasks inside it would run. If, however, there is a failure in the standard block, then the rescue block will perform its actions, and then the playbook will continue to process.
  -  You can't loop over a block. I can indeed still use loops on the task inside of a block, but I can't loop over a whole block itself.

``` yaml title="templates.yml"
---
- name: Add nginx webserver to a Rocky host
  hosts: LL-Test
  become: true
  vars:
    owner_name: Test
    web_path: /home/webserver
  tasks:
    - name: Block for Rocky hosts
      when: "'Rocky' in ansible_facts['distributions']"
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
      when: "'Ubuntu' in ansible_facts['distributions']"
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

     - name: Create home/webserver directory
       ansible.builtin.file:
         path: "{{ web_path }}"
         state: directory
         owner: nginx
         group: nginx
         mode: '0755'

     - name: Add index.html to webserver directory
       ansible.builtin.template:
         src: templates/index.html.j2 #Save HTML files with extention j2 and use the variables in the HTML file like {{ owner_name }}
         dest: "{{ web_path }}/index.html"
         owner: nginx
         group: nginx
         mode: '0644'

     - name: Add nginx config based on the template
       ansible.builtin.template:
         src: templates/nginx.config.j2 #
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

- You can use the Template Plugin: lookup('ansible.builtin.template','PathToTemplate.j2')
``` yaml
- name: Display template plugin
  ansible.builtin.debug:
    msg: "{{ lookup('ansible.builtin.template', './display-template.j2') }}"
```

- Handlers : 
  - If there were 10 tasks that all require a restart of the Nginx web server? That would mean there would need to be 10 additional tasks following each of those, ensuring the restart is completed.
  - This is where handlers come in. You can add a task level parameter to notify. Then reference the name of the task down in the handler section.
  - What this will do is that if a task with a notify parameter has a changed condition, Ansible will wait until all tasks have completed. Then it will call the handler a single time. This means you only require the single handler, and only call it once, no matter how many tasks activate the handler.
    
``` yaml title="handlers.yml"
---
- name: Add nginx webserver to a Rocky host
  hosts: LL-Test
  become: true
  vars:
    owner_name: Test
    web_path: /home/webserver
  tasks:
    - name: Block for Rocky hosts
      when: "'Rocky' in ansible_facts['distributions']"
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
      when: "'Ubuntu' in ansible_facts['distributions']"
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
        - name: Create home/webserver directory
          ansible.builtin.file:
            path: "{{ web_path }}"
            state: directory
            owner: nginx
            group: nginx
            mode: '0755'
        - name: Add index.html to webserver directory
          ansible.builtin.template:
            src: templates/index.html.j2 #Save HTML files with extention j2 and use the variables in the HTML file like {{ owner_name }}
            dest: "{{ web_path }}/index.html"
            owner: nginx
            group: nginx
            mode: '0644'
        - name: Add nginx config based on the template
          ansible.builtin.template:
            src: templates/nginx.config.j2 #
            dest: /etc/nginx/nginx.conf
            owner: nginx
            group: nginx
            mode: '0644'
          register: nginx_config
          notify: Restart nginx # Handler name

      handlers:  
        - name: Restart nginx
          when: nginx_config.changed
          ansible.builtin.service:
            name: nginx
            state: restarted
```

- Tags : 
  - Allow the admins to mark specific tasks in a playbook with metadata. Essentially you can assign special words, also known as tags.
  - When you call a playbook, you can instruct it to run tasks with only certain tags, you can specify tags to ignore, or you can do a combination of the two.
  - This could be useful if you have some long running playbooks and you only need to run certain portions.
    
``` yaml title="tags.yml"
---
- name: Add nginx webserver to a Rocky host
  hosts: LL-Test
  become: true
  vars:
    owner_name: Test
    web_path: /home/webserver
  tasks:
    - name: Block for Rocky hosts
      when: "'Rocky' in ansible_facts['distributions']"
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
      tags:
        - install
    - name: Block for Ubuntu hosts
      when: "'Ubuntu' in ansible_facts['distributions']"
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
      tags:
        - install

        - name: Create home/webserver directory
          ansible.builtin.file:
            path: "{{ web_path }}"
            state: directory
            owner: nginx
            group: nginx
            mode: '0755'
      tags:
        - configure

        - name: Add index.html to webserver directory
          ansible.builtin.template:
            src: templates/index.html.j2 #Save HTML files with extention j2 and use the variables in the HTML file like {{ owner_name }}
            dest: "{{ web_path }}/index.html"
            owner: nginx
            group: nginx
            mode: '0644'
        - name: Add nginx config based on the template
          ansible.builtin.template:
            src: templates/nginx.config.j2
            dest: /etc/nginx/nginx.conf
            owner: nginx
            group: nginx
            mode: '0644'
          register: nginx_config
          notify: Restart nginx # Handler name

      handlers:  
        - name: Restart nginx
          when: nginx_config.changed
          ansible.builtin.service:
            name: nginx
            state: restarted
```
> ansible-playbook -i inventory tags.yml --tags "install"
> ansible-playbook -i inventory tags.yml --skip-tags "configure"

- Execution mode
    - Run : is what is typically used and what I've used thus far. 
    - Check : will simulate automation changes rather than actually making changes.
      - When check mode is enabled, this task will never actually run, which means any future tasks that depend on that task information will fail to run properly.
      - To work around this limitation on that specific info gathering task, add the **check_mode:false** task level parameter. With this in place, even if the playbook is executed in check mode, this task will still execute.
> ansible-playbook -i inventory tags.yml --check

``` yaml title="assert.yml"
---
- name: Check RAM on a Rocky host
  hosts: LL-Test
  vars:
  tasks:
    - name: Display ansible_memtotal_mb
      ansible.builtin.debug:
        var: ansible_memtotal_mb
    - name: Assert if RAM is at least 10GB
      ansible.builtin.assert:
        that: ansible_memtotal_mb > 10000
        msg: RAM is less than 10GB
      
```
- This is a task level parameter, so need to be in line with the module name.
> changed_when: false </br>
> failed_when: false

- Nested loop : 
  - The best way to accomplish a nested loop is via a task file.

- Dynamic Inventory :
  -   It's a Python script that runs and generates an inventory from an external source.
  -   This source could be something as simple as a CSV file that is consumed, but more commonly, it's something like ServiceNow, VMware vCenter, AWS, or GCP. 
``` yaml title="dynamic-inventory.yml"
---
- name: Dynamic Inventory
  hosts: all
  gather_facts: all
  vars:
  tasks:
    - name: Print out the list of inventory hosts
      ansible.builtin.debug:
        var: groups['all']
      run_once: true
```
``` py title="custom_inventory.py"
#!/usr/bin/env python
#test import script
import json

# Specify the path to the JSON payload file
json_payload_file = 'files/json-payload'

# Read the JSON payload from the file
with open(json_payload_file, 'r') as file:
    json_payload = file.read()

# Parse the JSON payload
inventory_data = json.loads(json_payload)

# Initialize inventory data structures
ansible_inventory = {
    '_meta': {
        'hostvars': {}
    },
    'all': {
        'hosts': [],
        'vars': {
            # You can define global variables here
        }
    }
}

# Initialize group dictionaries for each OS
os_groups = {}

# Initialize group dictionaries for each location
location_groups = {}

# Process each host in the JSON payload
for host in inventory_data['hosts']:
    host_id = host['id']
    hostname = host['hostname']
    ip_address = host['ip_address']
    status = host['status']
    os = host['os']
    location = host['location']
    owner = host['owner']

    # Add the host to the 'all' group
    ansible_inventory['all']['hosts'].append(hostname)

    # Create host-specific variables
    host_vars = {
        'ansible_host': ip_address,
        'status': status,
        'os': os,
        'location': location,
        'owner': owner
        # Add more variables as needed
    }

    # Add the host variables to the '_meta' dictionary
    ansible_inventory['_meta']['hostvars'][hostname] = host_vars

    # Add the host to the corresponding OS group
    if os not in os_groups:
        os_groups[os] = {
            'hosts': []
        }
    os_groups[os]['hosts'].append(hostname)

    # Add the host to the corresponding location group
    if location not in location_groups:
        location_groups[location] = {
            'hosts': []
        }
    location_groups[location]['hosts'].append(hostname)

# Add the OS groups to the inventory
ansible_inventory.update(os_groups)

# Add the location groups to the inventory
ansible_inventory.update(location_groups)

# Print the inventory in JSON format
print(json.dumps(ansible_inventory, indent=4))
```
``` json title="json-payload.json"
{
  "hosts": [
    {
      "id": 1,
      "hostname": "host1.example.com",
      "ip_address": "192.168.1.101",
      "status": "active",
      "os": "Linux",
      "location": "dal-dc1",
      "owner": "Admin User"
    },
    {
      "id": 2,
      "hostname": "host2.example.com",
      "ip_address": "192.168.1.102",
      "status": "inactive",
      "os": "Windows",
      "location": "dal-dc1",
      "owner": "User 1"
    },
    {
      "id": 3,
      "hostname": "host3.example.com",
      "ip_address": "192.168.1.103",
      "status": "active",
      "os": "Linux",
      "location": "dal-dc1",
      "owner": "User 2"
    },
    {
      "id": 4,
      "hostname": "host4.example.com",
      "ip_address": "192.168.1.104",
      "status": "active",
      "os": "Windows",
      "location": "dal-dc1",
      "owner": "User 3"
    },
    {
      "id": 5,
      "hostname": "host5.example.com",
      "ip_address": "192.168.1.105",
      "status": "inactive",
      "os": "Linux",
      "location": "dal-dc1",
      "owner": "User 4"
    },
    {
      "id": 6,
      "hostname": "host6.example.com",
      "ip_address": "192.168.1.106",
      "status": "active",
      "os": "Linux",
      "location": "vir-dc1",
      "owner": "Admin User"
    },
    {
      "id": 7,
      "hostname": "host7.example.com",
      "ip_address": "192.168.1.107",
      "status": "inactive",
      "os": "Windows",
      "location": "vir-dc1",
      "owner": "User 1"
    },
    {
      "id": 8,
      "hostname": "host8.example.com",
      "ip_address": "192.168.1.108",
      "status": "active",
      "os": "Linux",
      "location": "vir-dc1",
      "owner": "User 2"
    },
    {
      "id": 9,
      "hostname": "host9.example.com",
      "ip_address": "192.168.1.109",
      "status": "active",
      "os": "Windows",
      "location": "vir-dc1",
      "owner": "User 3"
    },
    {
      "id": 10,
      "hostname": "host10.example.com",
      "ip_address": "192.168.1.110",
      "status": "inactive",
      "os": "Linux",
      "location": "vir-dc1",
      "owner": "User 4"
    }
  ]
}
```
> ansible-playbook -i files/custom_inventory.py dynamic-inventory.yml

- Sample
  ``` yaml title="test.yml"
  ---
  - name: 
    hosts: LL-rocky9-01
    # gather_facts: false
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
  
      - name: Create a file with specific content idempotently if ther eis a 3 in the dir name
        when: "'3' in item"
        ansible.builtin.copy:
          dest: "{{ item }}/file.txt"
          content: "This is the content of the file"
        loop: "{{ new_dirs }}"
  
      - name: Collect the contents of the new_dirs
        ansible.builtin.shell: "ls -l {{ item }}"
        loop: "{{ new_dirs }}"
        register: ls_output
        changed_when: false
  
      - name: Display the contents of the new_dirs
        when: item.stdout | length > 8
        ansible.builtin.debug:
          # msg: "{{ item.stdout }}"
          var: item.stdout
        loop: "{{ ls_output.results }}"
  ```

- Roles : is a simple way to abstract simple or complex portions of a playbook for usability.
  - If you are familiar with programming, it's similar in concept to a function or a subroutine. Ansible Roles are formatted in a very specific way.
  - Roles are generally stored in a directory called **Roles**.
  - Notice in the folders that there are a lot of files named **main.yml**.
  - If you are using these portions of a Role, which not all of these folders need to exist if you aren't using them, then you must use these main.yml files.
  - Ansible will always expect to see these files when these directories are used. Keep in mind, these files only contain the direct information for each section.
  ``` shel
  mkdir roles 
  ```
  ``` shel
  cd roles 
  ```
  ``` shel
  ansible-galaxy init nginx
  ```
  ``` yaml title="roles\nginx\defaults\main.yml"
  ---
  # defaults file for nginx
  owner_name: Greg Sowell
  web_path: /home/webserver
  ```
  ``` yaml title="roles\nginx\handlers\main.yml"
  ---
  # handlers file for nginx
  - name: Restart nginx
    when: nginx_config.changed
    ansible.builtin.service:
      name: nginx
      state: restarted
  ```
  ``` yaml title="roles\nginx\tasks\main.yml"
  ---
  # tasks file for nginx
  - name: Block for rocky hosts
    when: "'Rocky' in ansible_facts['distribution']"
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
  
  - name: Block for ubuntu hosts
    when: "'Ubuntu' in ansible_facts['distribution']"
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
  
  - name: Create /home/webserver directory
    ansible.builtin.file:
      path: "{{ web_path }}"
      state: directory
      owner: nginx
      group: nginx
      mode: '0755'
  
  - name: Add index.html to webserver directory
    ansible.builtin.template:
      src: templates/index.html.j2
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
    notify: Restart nginx
  ```
  ``` yaml title="nginx-role.yml"
  - name: Add/Configure nginx webserver on a Rocky host via Roles
    hosts: LL-Rocky9-01
    become: true
    vars:
    Roles:
      - nginx
  ``` 
  > ansible-playbook -i inventory nginx-role.yml
  - a better way to run Role. Ansible runs everything in the roles section first. Then it will complete everything in the tasks section. What if I wanted to run some tasks first, then roles, then some more tasks. Technically, you can create a pre_tasks section that will run first. 
  ``` yaml title="nginx-role.yml"
  - name: Add/Configure nginx webserver on a Rocky host via Roles
    hosts: LL-Rocky9-01
    become: true
    vars:
    # Roles:
    #   - nginx
      tasks:
        - name: Run nginx Role
        ansible.builtin.include_role:
          name: nginx
  ```
  -  Role Search Path
      1. local roles directory, which is relative to the playbook
      2. ANSIBLE_ROLES_PATH environment variables
      3. roles_path in the ansible.cfg file
      4. global system-wide /etc/ansible/roles directory
      5. hidden Ansible Galaxy directory. Likely the simplest thing to do is to drop your roles in a local roles folder
         
  -  Acquire Roles
      1. Git Clone
      2. Galaxy (ansible.galaxy.com)
         ``` sh
         ansible-galaxy install <role_name>
         ```
  -  Reqirements File
      1. List roles required
      2. ansible-galaxy install -r requirements.yml

- Secrets : encrypt secrets in a file via a method called **vaulting**. When you vault a file, it stays encrypted until you decide to decrypt it for use. It's most common to execute a playbook with an additional command line parameter so that it will decrypt the file in memory and pull in my secrets.
  - Do not store vaulted files in a public git repository just on the off chance it could be decrypted in the future
  ``` yaml title="files/secrets.yml"
  ---
  super_secret: Greg
  ```
  ``` sh
  ansible-vault encrypt secrets.yml
  ```
  ``` sh
  ansible-vault decrypt secrets.yml
  ```
  ``` yaml title="vault.yml"
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
  ```
  ansible-plabook -i inventory vault.yml --ask-vault-pass
  ```

## Windows Hosts
- Connections via WinRM
- Patch Management can be simplified even when using WSUS, Intune, or other vendor products
- Configuration management and app deployment, admin tasks like
  - Active Directory configurations
  - Config/Verify security policies
  - PowerShell integration
    
``` yaml title="windows.yml"
---
- name: Perform windows updates
hosts: windows-01
gather_facts: false
# become: true
vars_prompt:
  - name: "ansible_ssh_pass"
    prompt: Windows password
    private: true

tasks:
  - name: Update the windows server
    ansible.windows.win_updates:
      category_names:
        # - SecurityUpdates
        - CriticalUpdates
        # - DefinitionUpdates
        # - UpdateRollups
        # - FeaturePacks
        # - ServicePacks
        # - UpdateRollups
        # - Updates
      state: searched # if any update available
      # reboot: yes
      # reboot_timeout: 3600
    register: update_output

  # - name: Print out the results
  #   ansible.builtin.debug:
  #     var: update_output

  # - name: Clear all event logs
  #   win_shell: wevtutil el | ForEach-Object { wevtutil cl $_ }
```
> ansible-playbook -i inventory windows.yml

## Network Hosts
``` yaml title="network.yml"
---
- name: SNMP updates on a Cisco device
  hosts: sw1
  gather_facts: false
  # ensure you have paramiko installed on the ansible server
  # pip3 install --upgrade paramiko ansible-pylibssh
  # update-crypto-policies --set LEGACY
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
          # - snmp-server community public group network-operator # should run full command
          # - snmp-server community private group network-admin

##### Resource Module Replace Start
    # - name: Replace snmp-server configurations
    #   cisco.nxos.nxos_snmp_server:
    #     config:
    #       communities:
    #         - name: public
    #           group: network-operator
    #         - name: private
    #           group: network-admin
    #       users:
    #         auth:
    #           - user: admin
    #             authentication:
    #               engine_id: "128:0:0:9:3:0:80:86:135:240:25"
    #               algorithm: md5
    #               password: "0xe2bb3e2405ce601c5c3993262270ace5"
    #               localized_key: true
    #               priv:
    #                 privacy_password: "0xe2bb3e2405ce601c5c3993262270ace5"
    #     state: replaced
##### Resource Module Replace End

##### Parsed Config Start #####
    # - name: Parse externally provided snmp-server configuration
    #   cisco.nxos.nxos_snmp_server:
    #     running_config: "{{ parse_me }}"
    #     state: parsed
    #   register: parsed_config

    # - name: Debug parsed_config
    #   ansible.builtin.debug:
    #     var: parsed_config

    # - name: Replace snmp-server configurations
    #   cisco.nxos.nxos_snmp_server:
    #     config: "{{ parsed_config.parsed }}"
    #     state: replaced
##### Parsed Config End #####

    - name: Find current SNMP configuration
      cisco.nxos.nxos_command:
        commands: show run | inc snmp-ser
      register: snmp_output

    - name: Display current SNMP configuration
      ansible.builtin.debug:
        var: snmp_output.stdout_lines
```
> You can store, in the change configuration, the exact CLI commands you want to use, only change that file, manage that file, modify that file, and use the automation to apply it.

## APIs
You can always use the URI module, which is exactly what I plan to demonstrate. 
``` yaml title="api.yml"
- name: Fetch current weather data for College Station, Texas
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

    # - name: Print chance of rain tomorrow
    #   ansible.builtin.debug:
    #     msg: >-
    #       Tomorrows chance of rain(percentage):
    #       Morning: {{ (forecast_response.json.properties.periods | selectattr('number', 'equalto', 3) | list).0.probabilityOfPrecipitation.value | default(0, true) }} and
    #       Afternoon: {{ (forecast_response.json.properties.periods | selectattr('number', 'equalto', 4) | list).0.probabilityOfPrecipitation.value | default(0, true) }}
```
