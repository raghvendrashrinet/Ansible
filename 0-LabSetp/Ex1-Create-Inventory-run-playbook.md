### Inventrory file
`inventory`
```ini
stapp03 ansible_host=stappo3 ansible_user=banner ansible_ssh_pass=BigB0ss
```

```
┌──────────────┬───────────────────────────────┬───────────────────┐
│ stapp01      │ ansible_host=stapp01          │ ansible_user=tony │
└──────────────┴───────────────────────────────┴───────────────────┘
      │                    │
      │                    │
      ▼                    ▼
Inventory Hostname     Real SSH Host
```

2.  copy ssh-copy-id to target server
   `ssh-copy-id -i ../.ssh/id_ed25519.pub banner@stapp03 `
3. Run test ping module
   ` ansible server -m ping -i inventory.ini `
4.  Playbook
   `playbook.yaml`
```
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: Install httpd package    
      yum: 
        name: httpd 
        state: installed
    
    - name: Start service httpd
      service:
        name: httpd
        state: started
```
5. Run playbook
   ```
    ansible-playbook -i inventory.ini playbook.yml
   ```
> [!NOTE]
> By default, when become: yes is enabled, Ansible automatically defaults to root. So explicitly declaring
> become_user: root simply tells Ansible:
>
> Connect to the target host as the SSH user (e.g., banner), but execute these tasks as root using sudo.
