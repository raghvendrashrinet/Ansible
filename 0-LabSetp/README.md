#### Step 1: Create a docker-compose.yml File
In the root of your Ansible repository, create a docker-compose.yml file:

```YAML
version: '3.8'

services:
  # Ansible Control Node
  ansible-controller:
    image: ubuntu:22.04
    container_name: ansible-controller
    volumes:
      - .:/ansible
      - /var/run/docker.sock:/var/run/docker.sock
    working_dir: /ansible
    command: >
      sh -c "apt-get update && 
             apt-get install -y python3 python3-pip docker.io && 
             pip install ansible && 
             tail -f /dev/null"
    networks:
      - ansible-net

  # Target Node 1
  target1:
    image: ubuntu:22.04
    container_name: target1
    command: sh -c "apt-get update && apt-get install -y python3 && tail -f /dev/null"
    networks:
      - ansible-net

  # Target Node 2
  target2:
    image: ubuntu:22.04
    container_name: target2
    command: sh -c "apt-get update && apt-get install -y python3 && tail -f /dev/null"
    networks:
      - ansible-net

networks:
  ansible-net:
    driver: bridge
```
#### Step 2: Configure inventory/hosts
Update your Ansible inventory file to target these containers using the docker connection plugin:

```Ini, TOML
[webservers]
target1 ansible_connection=docker
target2 ansible_connection=docker
```
#### Step 3: Manage Your Lab Lifecycle
Now you can bring your entire test environment up or tear it down with single commands:

Start the dummy servers:

```Bash
docker compose up -d
```
Test connectivity with Ansible:

```Bash
ansible webservers -m ping
```
Stop and remove the lab when done:
```Bash
docker compose down
```
#### Step 4: Run a Test Playbook
Create a test playbook site.yml:

```YAML
---
- name: Test Ansible on Docker Targets
  hosts: webservers
  tasks:
    - name: Print hello message
      ansible.builtin.debug:
        msg: "Hello from {{ inventory_hostname }}!"

    - name: Ensure Nginx is installed
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: yes
```
Run the playbook against your live Compose stack:

```Bash
ansible-playbook site.yml
```
