
Excercise 
- install httpd web server on all app servers. Additionally, make sure its service should up and running.
- Using blockinfile Ansible module add some content in /var/www/html/index.html file. Below is the content:

```
Welcome to XfusionCorp!

This is  Nautilus sample file, created using Ansible!

Please do not modify this file manually!
```

- The /var/www/html/index.html file's user and group owner should be apache on all app servers.

- The /var/www/html/index.html file's permissions should be 0644 on all app servers.



`playbook.yml`

```yaml
---
- name: Install httpd
  hosts: all
  become: yes  # Uses sudo to install the package

  tasks:
    - name: Install Httpd
      ansible.builtin.yum:
        name: httpd
        state: present
    - name: Ensure httpd service is started and enabled on boot
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes
    - name: Create or update index.html with specific block content
      ansible.builtin.blockinfile:
        path: /var/www/html/index.html
        create: yes               # Creates the file if it does not exist yet
        owner: apache             # Sets web server ownership
        group: apache
        mode: '0644'              # Standard readable permissions
        block: |
          <!DOCTYPE html>
          <html>
          <head>

          </head>
          <body>
            Welcome to XfusionCorp!

           This is  Nautilus sample file, created using Ansible!

           Please do not modify this file manually!
          </body>
          </html>

```
