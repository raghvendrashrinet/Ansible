# Ansible Task: Configure httpd Role with Jinja2 Template

## Task Description
The DevOps team is developing an Ansible role for `httpd` installation and configuration. Complete the remaining steps to deploy a dynamic `index.html` file using a Jinja2 template targeting **App Server 2**.

### Requirements
1. **Playbook Update:** Update `~/ansible/playbook.yml` to execute the `httpd` role on **App Server 2**.
2. **Jinja2 Template:** Create a template at `/home/thor/ansible/role/httpd/templates/index.html.j2` containing:
   ```text
   This file was created using Ansible on {{ inventory_hostname }}
   ```
   Do not hardcode the server name; use the inventory_hostname Ansible variable.
3. Ansible Task: Add a task inside `/home/thor/ansible/role/httpd/tasks/main.yml` to deploy this template to `/var/www/html/index.html` on App Server 2.
4. Permissions & Ownership: Set `/var/www/html/index.html file permissions to 0777`. Set the user and group owner to the respective sudo user of the server (e.g., steve for stapp02).


---
#### 1. What is an Ansible Role?
An Ansible Role is a way to organize Ansible automation tasks into a standardized directory structure. Instead of putting everything into one huge file, a role breaks it down into folders:
- `tasks/main.yml`: Contains the list of actions to perform (e.g., install httpd, start service, copy files).
- `templates/`: Holds template files (like index.html.j2) that will be populated dynamically and sent to target servers.
- `defaults/`, `vars/`, `handlers/`: Store default variables, extra variables, or service triggers.

Using a role makes your Ansible code reusable and clean.

#### 2. What is Jinja2?
Jinja2 is a templating engine used by Python and Ansible. It allows you to write files with dynamic variables instead of static text.
- Standard text file: `This file was created on stapp02`
- Jinja2 template file (index.html.j2): This file was created using Ansible on` {{ inventory_hostname }}`
- When Ansible runs this template against App Server 2 (stapp02), Jinja replaces` {{ inventory_hostname }} `with `stapp02` automatically.

#### 3. What is inventory_hostname?
`inventory_hostname` is a built-in Ansible variable containing the name of the server currently being configured as defined in your Ansible inventory file.

---

#### Step 2: Breaking Down the Requirements
- Target Server: App Server 2 (stapp02).
- `Playbook (~/ansible/playbook.yml)`: Must be updated to target stapp02 (or App Server 2's group/hostname) and apply the httpd role.
- `Jinja2 Template` (/home/thor/ansible/role/httpd/templates/index.html.j2):
   - Path: `/home/thor/ansible/role/httpd/templates/index.html.j2`
   - Exact Content: This file was created using Ansible on `{{ inventory_hostname }}`

- Ansible Task (/home/thor/ansible/role/httpd/tasks/main.yml):
  - Use the ansible.builtin.template module (or template) to deploy index.html.j2 to /var/www/html/index.html.
  - Set permissions to 0777.
  - Set owner and group to the respective sudo user of the server (for stapp02, check inventory or server details; typically steve for stapp02, but double check using cat inventory or Details of all Users and Servers).
---
Execution
#### Create the Template directory and file
- Create the templates directory if it doesn't already exist:

```Bash
mkdir -p /home/thor/ansible/role/httpd/templates
```

- Create the file `/home/thor/ansible/role/httpd/templates/index.html.j2` with this content:
```
This file was created using Ansible on {{ inventory_hostname }}
```

- Add the Template Task to the Role
Edit /home/thor/ansible/role/httpd/tasks/main.yml and add a template task at the end of the file.

Assuming the sudo user for App Server 2 (stapp02) is steve (confirm from inventory or user details button):
```
- name: Copy index.html template to web root
  ansible.builtin.template:
    src: index.html.j2
    dest: /var/www/html/index.html
    mode: '0777'
    owner: steve
    group: steve
```

- Update playbook.yml
Edit ~/ansible/playbook.yml to run the httpd role on App Server 2:

```YAML
- name: Install and configure httpd on App Server 2
  hosts: stapp02
  become: yes
  roles:
    - httpd
```

Test:  
```
cd ~/ansible
ansible-playbook -i inventory playbook.yml
```
