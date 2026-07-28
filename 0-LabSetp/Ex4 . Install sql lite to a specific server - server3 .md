## Install sql lite to a specific server - server3 only using ansible yum module

#### This command list inventory as seen by ansible 
- Check how the host is seen by ansible from inventory file
```
ansible-inventory -i inventory --list
```
- `inventory` file 
	```ini
		stapp01 ansible_user=tony     ansible_ssh_pass=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
	stapp02 ansible_user=steve    ansible_ssh_pass=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
	stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
	```
	
- `playbook.yml`
```yaml
  ---
    - hosts: all
     become: yes
     become_user: root
     tasks:
       - name: Ensure SQLite package is installed
         ansible.builtin.yum:
           name: sqlite
           state: present
```
	
- Run:
	
  - To run on all servers
    
	   `ansible-playbook -i inventory playbook.yml`
  - To run on perticular host server
      
     `ansible-playbook -i inventory playbook.yml --limit stapp03`



