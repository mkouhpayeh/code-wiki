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
