### Ansible bootstrap playbook 
you can run once against a fresh server (using password authentication the first time). It will create the automation user, install your public key, and configure passwordless sudo — so afterward you can manage the host entirely with key‑based Ansible access.  

`bootstrap.yaml`
```yaml
---
- name: Bootstrap target servers for Ansible
  hosts: all
  become: true
  vars:
    ansible_user_name: ansible
    ansible_pub_key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... ansible-control-node"

  tasks:
    - name: Ensure user exists
      user:
        name: "{{ ansible_user_name }}"
        comment: "Ansible Automation User"
        groups: sudo,wheel
        shell: /bin/bash
        create_home: yes

    - name: Add authorized key for Ansible user
      authorized_key:
        user: "{{ ansible_user_name }}"
        key: "{{ ansible_pub_key }}"
        state: present

    - name: Allow passwordless sudo for Ansible user
      copy:
        dest: "/etc/sudoers.d/{{ ansible_user_name }}"
        content: "{{ ansible_user_name }} ALL=(ALL) NOPASSWD:ALL\n"
        owner: root
        group: root
        mode: '0440'

```
#### How It Works
- user → Creates the ansible user with home directory, shell, and admin groups.
- authorized_key → Installs your public key into ~/.ssh/authorized_keys.
- copy → Drops a sudoers file granting passwordless sudo.

`Run:`
```
ansible-playbook -i inventory bootstrap.yml --ask-pass
```
From here on, Ansible connects with keys and no password prompts.
