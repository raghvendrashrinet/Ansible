
#### Ex Using Inleinfile Module 

- Install httpd web server on all app servers, and make sure its service is up and running.

- Create a file /var/www/html/index.html with content:

```
This is a Nautilus sample file, created using Ansible!
```

- Using lineinfile Ansible module add some more content in /var/www/html/index.html file. Below is the content:
```
Welcome to Nautilus Group!
```
Also make sure this new line is added at the top of the file.
- The /var/www/html/index.html file's user and group owner should be apache on all app servers.
- The /var/www/html/index.html file's permissions should be 0755 on all app servers.



-  `playbook.yml`
```
---
- name: Setup httpd and deploy index.html on all app servers
  hosts: all
  become: yes

  tasks:
    - name: Install httpd package
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Ensure httpd is started and enabled
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes

    - name: Create /var/www/html/index.html with base content, permissions, and ownership
      ansible.builtin.copy:
        content: "This is a Nautilus sample file, created using Ansible!\n"
        dest: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0777'

    - name: Add Welcome line at the top of the file
      ansible.builtin.lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to Nautilus Group!"
        insertbefore: BOF
        owner: apache
        group: apache
        mode: '0777'
```

- Verify

```
ansible -i inventory stapp01 -m ansible.builtin.stat -a "path=/var/www/html/index.html" | grep -E "mode|owner|group"

```
