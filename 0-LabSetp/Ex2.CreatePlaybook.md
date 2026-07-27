## Create a playbook /home/thor/ansible/playbook.yml. 
- Include a task to create an empty file /tmp/file.txt on App Server 1.
##### Invertory file 
```ini
stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
- Verify
` ansible all -i inventory -m ping`

#### Create playbook
`playbook.yaml`
```yaml
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Create file.txt in tmp folder    
      ansible.builtin.file:
        path: /tmp/file.txt
        state: touch
        mode: '0644' # Sets read/write permissions
        owner: root   # Sets file owner
        group: root
```

####
Run:  
```
 ansible-playbook -i inventory playbook.yml
```
