### In Ansible, a role is a structured, reusable folder format that lets you bundle together your automation tasks,variables, files, templates, and handlers.
Instead of writing one massive, messy playbook file with hundreds of lines of code, a role lets you break your automation down into neat, modular components. For example, you might create an nginx role, a database role, or a newman-tests role, 
and then mix and match them inside your master playbooks.

### The Standard Ansible Role Structure
When you create a role (usually by running the command ansible-galaxy role init my_role), Ansible builds a specific directory tree. 
Ansible automatically knows exactly where to look for code based on these folder names:

`ansible-galaxy role init my_role`
```
my_role/
├── defaults/    # Default variables for the role (lowest priority, easy to override)
├── vars/        # Strict variables specific to this role (higher priority)
├── tasks/       # The main list of actions/tasks to execute (main.yml)
├── files/       # Static files to copy to servers (like images or scripts)
├── templates/   # Jinja2 templates (like your config.conf.j2 file!)
├── handlers/    # Tasks triggered by other tasks (e.g., restarting a service)
├── meta/        # Information about the role (author, dependencies, supported OS)
└── README.md    # Documentation describing how to use the role

```

#### How it looks in practice
Let's look at how you would use an Ansible role to solve your exact problem from earlier: forcing the creation of /etc/myapp and deploying your Jinja2 template.

##### 1. Inside your role (my_role/tasks/main.yml):You write the tasks cleanly inside the role folder, completely independent of specific server addresses.

```yaml
---
# tasks file for my_role
- name: Force create the destination directory
  ansible.builtin.file:
    path: "{{ app_directory }}"
    state: directory
    mode: '0755'

- name: Deploy the dynamic dynamic configuration file
  ansible.builtin.template:
    src: config.conf.j2
    dest: "{{ app_directory }}/config.conf"
```
##### 2. Inside your playbook (deploy.yml):To apply those tasks to your target servers, you simply reference the role name in your master playbook file.

```yaml
---
- name: Set up application infrastructure
  hosts: webservers
  become: yes  # Runs as root to write to /etc
  vars:
    app_directory: "/etc/myapp"  # Overrides or sets your directory variable
  
  roles:
    - my_role  # Ansible instantly loads and runs everything inside the role
```
jinja2 template : is inside templates/config.conf.j2
Run playbook
```
~> ansible-playbook -i inventory playbook.yml


## because of role field it calls the playbook in task/playbook.yml and executes 
```
