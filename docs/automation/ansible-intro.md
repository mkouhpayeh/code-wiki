# Ansible

*Written in Python. The Ansible controller requires Python.*  
Ansible follows the principle of **idempotence** — you can run automation repeatedly and it only makes changes when necessary.  
Ansible uses the **Jinja2 templating engine** for variable replacement, loops, conditionals, and data manipulation.


---

## Keywords

- **Community**  
  The easiest way for most admins to run Ansible. It contains Ansible binaries and many commonly used components packaged and ready to go.

- **Core**  
  A minimal distribution containing only essential components. Developers or admins who want a lightweight install use this; additional components may be required to automate most hosts.

- **Navigator**  
  Builds on top of Ansible Core by allowing executions inside containers called *Execution Environments* (EEs).

- **Container**  
  A portable unit bundling code, runtime, libraries, and dependencies to run an application consistently across environments.

- **API**  
  Application Programming Interface.

- **Playbooks**  
  Scripts consumed by the Ansible binary to perform automation.
  1. A playbook tells each *play* where to run and what to do.
  2. A playbook can contain multiple plays (different sets of hosts).
  3. The `name` field is optional but highly recommended — it documents intent and appears in output.

- **Tasks**  
  Ordered list of discrete automation steps executed in sequence. By default Ansible completes the first task for all hosts before moving to the next.

- **Module**  
  A piece of code (usually Python or PowerShell) that performs an automation action.

- **Collection**  
  An archive (tar) that installs modules, roles, plugins, playbooks, docs, and other objects. Use the Fully Qualified Collection Name (FQCN) to reference modules.

- **Inventory**  
  A list of hosts you can operate against. Think of the inventory as a menu — the `hosts` declaration in a playbook is the specific order you pick from that menu. Inventories are typically stored as INI or YAML files.

- **YAML**  
  Defines layout and structure — indentation matters:
  - Use spaces, not tabs.
  - Tabs will break playbooks.
  - Two spaces is a common indentation rule.
  - Many mistakes are due to incorrect spacing.

- **Variables**  
  Important for flexibility and fact gathering. Ansible has many precedence levels (22 levels). Use variables as `{{ variable_name }}`.

- **AWX**  
  A web UI and job controller for Ansible. AWX provides:
  - Role-based access control (RBAC)
  - A GUI and REST API
  - Job logging and execution control

- **Python Virtual Environment (PVE)**  
  Lets you manage different Python package versions per environment. When managing many dependencies, consider Execution Environments and Ansible Navigator instead.

---

## Install

```bash
# Install Python (Fedora/RHEL example)
sudo dnf install -y python3

# Install pip
sudo dnf install -y python3-pip

# Install Ansible
pip install ansible

# Update Ansible
pip install --upgrade ansible

# Verify
ansible --version
```

> If you have SSH issues from macOS hosts, you may need Paramiko:
```bash
pip install paramiko
```

---

## Commands

- `pwd` — Present Working Directory  
- `ls` — List files  
- `clear` — Clear terminal  
- `--ask-pass` — Prompt for SSH password  
- `--ask-become-pass` — Prompt for privilege escalation (become) password

---

## Examples

1. Gather facts and display them
```yaml
---
- name: Gather facts and display them
  hosts: LL-Test
  tasks:
    - name: Display all gathered facts
      ansible.builtin.debug:
        var: ansible_facts

    - name: Display the currently running kernel version and distro
      ansible.builtin.debug:
        msg: "The kernel version is {{ ansible_facts.kernel }} and the distribution is {{ ansible_facts.distribution }}"
```
Run:
```bash
ansible-playbook -i inventory gather-facts.yml
```

---

2. Quick webserver install and config
```yaml
---
- name: Quick webserver install and config
  hosts: LL-Test
  gather_facts: false
  become: true
  tasks:
    - name: Install apache webserver
      ansible.builtin.package:
        name: httpd
        state: present

    - name: Start and enable apache
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Write a simple index.html file
      ansible.builtin.lineinfile:
        path: /var/www/html/index.html
        line: "<h1>Hello world!</h1>"
        create: true
```
Run:
```bash
ansible-playbook -i inventory webserver.yml
# then test with:
curl http://127.0.0.1
```

---

3. Vars, conditionals and loops
```yaml
---
- name: Vars, conditionals and loops
  hosts: LL-Test
  gather_facts: false
  vars:
    nums:
      - 1
      - 10
      - 20
      - 25
      - 30
  tasks:
    - name: Display any numbers larger than 10
      ansible.builtin.debug:
        msg: "{{ item }}"
      loop: "{{ nums }}"
      when: item > 10
```

---

## Idempotence output (playbook run statuses)

- **OK** — Task completed successfully and no change was needed.  
- **Changed** — Task completed and made a change on the host.  
- **Failed** — Task failed for this host (error details are shown).  
- **Skipping** — Condition not met, so task was skipped.

---

## Installing Collections (quick steps)

1. Install a collection:
```bash
ansible-galaxy collection install <namespace.collection_name>
```
2. Check repository `requirements.txt` for Python dependencies. If a `requirements.txt` URL is provided, you can install its Python deps:
```bash
pip install -r <requirements.txt-raw-url>
```

---


