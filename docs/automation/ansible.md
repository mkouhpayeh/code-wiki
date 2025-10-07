# Ansible
Written in Python. Ansible controller requires Python. </br>
It's the concept that says, I can rerun a piece of automation as many times as I want, and it will only make a change when necessary. </br> 

## Keywords
- Community : is what most admins should install if they want to directly run Ansible. </br> 
  It contains the Ansible binaries, along with a slew of some of the most commonly used components, all packaged up and ready to go. This is the easiest way to get started.
- Core : is a stripped-down version that really only contains the essential components for Ansible to function. This means that to automate against most hosts, additional components will need to be installed. </br> 
  This is most often used by developers or admins that want to only install the absolutely required pieces.
- Naviagtor : It builds on top of Ansible Core by allowing you to execute your automations inside of containers, referring to them as Execution Environments, or EEs. 
- Container : is a lightweight, portable, and self-sufficient unit that includes all the necessary code, runtime, libraries, and dependencies to run an application consistently across different computing environments.
  That's a complicated definition.
- API : Application Programming Interface
- Playbooks : are the scripts that the Ansible binary will consume to perform automations. </br> 
    1.  A playbook tells each player where to go and what to do when they get there. The concept is the same in Ansible.
    2.  While a playbook can consist of multiple plays, where you define a new set of hosts to operate against.
    3.  Name is technically optional, though it should always be used. This acts as documentation and also shows up in automation output so that it's easier to understand what's happening.
- Tasks : are an ordered list of discrete automations that will be performed in sequence. Ansible will start at the top and, by default, complete the first task for all hosts before moving on to the next.
- Module : is a reference to a piece of code, generally written in Python or PowerShell, that will perform some piece of automation.
- Collection : is really just an archive file in TAR format, much like a zip file, and when installed, it just places Python or PowerShell scripts on the Ansible server. Keeping this in mind, when I want to reference a         module, I should use the Fully Qualified Collection Name, or FQCN.
- Inventory : is a big list of all the hosts I could potentially operate against. So, the host portion of a playbook is truly just referencing hosts that live inside of an inventory. </br> 
  The inventory is as the menu at your favorite restaurant. The host portion of the playbook is just the very specific order you're going to be making. </br> 
  When executing a playbook, an admin will specify which inventory file to utilize for each run. Inventories, when working on the command line, generally are stored in a file in either INI format or in YAML format.
- YAML : It really dictates how things are laid out, and indentation is very important.
    1.  You use spaces, and not tabs.
    2.  Ansible will break if there are any tabs in your playbook.
    3.  When indenting, which is YAML way of showing precedence, the rule of thumb is generally two spaces.
    4.  Most of the mistakes an admin makes, junior or not, are due to spacing.
- Variables : are important for information gathering, as well as adding flexibility to playbooks. There are 22 levels of variable precedence in Ansible.
  use as {{ variable_name }}
- AWX : is an air traffic controller for Ansible. Essentially it says who can do what, when, and where with Ansible playbooks. It, first, has a nice graphical user interface, or GUI.
  AWXs have an application programming interface, which allows for communication from external systems.
  It provides the role-based access control, or RBAC.
  View details logs.
- Collection : is an archive file that contains modules, roles, plugins, playbooks, documentation, and other objects.
  Installing a collection isn't too bad. It's generally done by issuing the Ansible-Galaxy Collection install command.
  But I will also need to ensure that I have all the Python dependencies.
    1.  run in terminal : ansible-galaxy collection install xxxx
    2.  Go to Repo to find all requirements and find the "requirements.txt" file
    3.  Click "Raw" and copy the URL
    4.  pip install -r URL
- Python Virtual Environment (PVE) : is a way you can have different Python packages installed at different versions. </br> 
  It can get a little hairy maintaining them after a while, and in this case, I would look at upgrading to using Ansible Navigator in execution environments.

## Install
``` title="Install Python"
sudo dnf install -y python3
```
``` title="Install Python Package Manager"
sudo dnf install -y python3-pip
```
``` title="Install Ansible"
pip install ansible
```
``` title="Update Ansible"
pip install ansible -u
```
``` title="Verify Ansible"
ansible --version
```
>  if you are having issues to host via SSH with Macs, you may need to install ParaMiko with :
``` 
pip install paramiko
```

## Commands
- pwd : Presend Working Directory
- ls : list
- clear : clear Terminal
- --ask-pass : ask for the ssh password
- --ask-become-pass : privilege escalation in Ansible is referred to as "becoming.

- Sample
  ``` yaml title="gather-facts.yml"
    ---
    - name: Gather facts and display it
      hosts: LL-Test
      Tasks:
      - name: Display all gathered facts
        ansible.builtin.debug:
          var: ansible_facts
    
      - name: Display the currently running kernel version and distro
      ansible.builtin.debug
        msg: "The kernel version is {{ ansible_facts.kernel }} and the distribution is {{ ansible_facts.distribution }}"
  ```
  > ansible-playbook -i inventory gather-facts.yml

  ``` yaml title="webserver.yml"
  ---
  - name: Quick webserver install and config
    hosts: LL-Test
    gather-facts: false
    become: true
    tasks:
      - name: Install apache webserver
        ansible.builtin.package:
          name: httpd
          state: present

     - name: Start and Enable apache
       ansible.builtin.service:
         name: httpd
         state: started
         enabled: true

     - name: Write a simple index.html file
       ansible.builtin.ineinfile:
         path: /var/www/html/index.html
         line: "<h1>Hello world!</h1>"
         create: true
  ```
  > ansible-playbook -i inventory webserver.yml </br> 
  > curl http://127.0.0.1 </br>

  ``` yaml title="vcl.yml"
  ---
  - name: Vars, conditionals and loops
    hosts: LL-Test
    gather-facts: false
    vars:
      nums:
        - 1
        - 10
        - 20
        - 25
        - 30
    tasks:
      - name: Display any numbers larger than 10
        when: item > 10
        ansible.builtin.debug:
          msg: "{{ item }}"
        loop: "{{ nums }}"
  ```

## Idempotence Output
  - OK : means everything completed successfully, but the automation didn't need to make a change.
  - Changed : means everything completed successfully, and the automation made a change to the host.
  - Failed : indicates that the task failed to complete for this host, and it will usually give some indication as to why.
  - Skipping : will only come into play when a conditional is used. If the condition is not met, it will show the notice.
