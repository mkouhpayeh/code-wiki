# Ansible
Written in Python. Ansible controller requires Python.

## Keywords
- Community :  is what most admins should install if they want to directly run Ansible.
  It contains the Ansible binaries, along with a slew of some of the most commonly used components, all packaged up and ready to go. This is the easiest way to get started.
- Core : is a stripped-down version that really only contains the essential components for Ansible to function. This means that to automate against most hosts, additional components will need to be installed.
  This is most often used by developers or admins that want to only install the absolutely required pieces.
- Naviagtor : It builds on top of Ansible Core by allowing you to execute your automations inside of containers, referring to them as Execution Environments, or EEs. 
- Container : is a lightweight, portable, and self-sufficient unit that includes all the necessary code, runtime, libraries, and dependencies to run an application consistently across different computing environments.
  That's a complicated definition.
- API : Application Programming Interface
- Playbooks : are the scripts that the Ansible binary will consume to perform automations.
- YAML : It really dictates how things are laid out, and indentation is very important.
  - If you remember anything about YAML, let it be that you use spaces, and not tabs.
  - Ansible will break if there are any tabs in your playbook.
  - When indenting, which is YAML way of showing precedence, the rule of thumb is generally two spaces.
  - Most of the mistakes an admin makes, junior or not, are due to spacing. 
