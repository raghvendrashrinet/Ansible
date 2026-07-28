### Problem
1. Create an empty file blog.txt under /opt/itadmin/ directory on app server 1. Set some acl properties for this file. Using acl provide read '(r)' permissions to group tony (i.e entity is tony and etype is group).
2. Create an empty file story.txt under /opt/itadmin/ directory on app server 2. Set some acl properties for this file. Using acl provide read + write '(rw)' permissions to user steve (i.e entity is steve and etype is user).
3. Create an empty file media.txt under /opt/itadmin/ on app server 3. Set some acl properties for this file. Using acl provide read + write '(rw)' permissions to group banner (i.e entity is banner and etype is group).

### Playbook
`playbook.yml`
```yaml
---
- name: Configure Advanced File ACLs Across App Server Fleet
  hosts: all
  become: yes  # Modifying ACLs requires root privileges

  tasks:
    # -------------------------------------------------------------------------
    # APP SERVER 1 (stapp01) CONFIGURATION
    # -------------------------------------------------------------------------
    - name: Create empty blog.txt on app server 1
      ansible.builtin.file:
        path: /opt/itadmin/blog.txt
        state: touch
      when: inventory_hostname == 'stapp01'

    - name: Set group tony read ACL on blog.txt
      ansible.posix.acl:
        path: /opt/itadmin/blog.txt
        etype: group
        entity: tony
        permissions: r
        state: present
      when: inventory_hostname == 'stapp01'

    # -------------------------------------------------------------------------
    # APP SERVER 2 (stapp02) CONFIGURATION
    # -------------------------------------------------------------------------
    - name: Create empty story.txt on app server 2
      ansible.builtin.file:
        path: /opt/itadmin/story.txt
        state: touch
      when: inventory_hostname == 'stapp02'

    - name: Set user steve read/write ACL on story.txt
      ansible.posix.acl:
        path: /opt/itadmin/story.txt
        etype: user
        entity: steve
        permissions: rw
        state: present
      when: inventory_hostname == 'stapp02'

    # -------------------------------------------------------------------------
    # APP SERVER 3 (stapp03) CONFIGURATION
    # -------------------------------------------------------------------------
    - name: Create empty media.txt on app server 3
      ansible.builtin.file:
        path: /opt/itadmin/media.txt
        state: touch
      when: inventory_hostname == 'stapp03'

    - name: Set group banner read/write ACL on media.txt
      ansible.posix.acl:
        path: /opt/itadmin/media.txt
        etype: group
        entity: banner
        permissions: rw
        state: present
      when: inventory_hostname == 'stapp03'

```

### The Terminal Command (Run on All Servers)
```
ansible-playbook -i inventory playbook.yml
```
###  Verify Everything
```
# Verify Server 1
ansible -i inventory stapp01 -m ansible.builtin.command -a "getfacl /opt/itadmin/blog.txt"

# Verify Server 2
ansible -i inventory stapp02 -m ansible.builtin.command -a "getfacl /opt/itadmin/story.txt"

# Verify Server 3
ansible -i inventory stapp03 -m ansible.builtin.command -a "getfacl /opt/itadmin/media.txt"
```




