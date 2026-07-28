#### Create a playbook ~/playbook/playbook.yml to create a blank file /opt/data.txt on all app servers.
-  Set the permissions of the /opt/data.txt file to `0744.`
- Ensure the `user/group owner` of the /opt/data.txt file is` tony on app server 1, steve on app server 2 and banner on app server 3`.


- inventory file
```ini
[app_servers]
stapp01 ansible_user=tony owner_name=tony ansible_ssh_pass=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_user=steve owner_name=steve ansible_ssh_pass=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_user=banner owner_name=banner ansible_ssh_pass=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
Playbook.yaml

```yaml
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Create nfsshare.txt in home folder
      ansible.builtin.file:
        path: /home/nfsshare.txt
        state: touch
        mode: '0755'
        owner: "{{ owner_name }}"
        group: "{{ owner_name }}"

    - name: Get file stats
      ansible.builtin.stat:
        path: /home/nfsshare.txt
      register: file_info

    - name: Display file details
      ansible.builtin.debug:
        msg: >
          Owner: {{ file_info.stat.pw_name }},
          Group: {{ file_info.stat.gr_name }},
          Mode: {{ file_info.stat.mode }}
```
