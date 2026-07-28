### What is does?
When you add host_key_checking = False or -o StrictHostKeyChecking=no, you are telling your machine to blindly trust the server

1. In `ansible.cfg` file
```
[defaults]
host_key_checking = False
```

2. In inventory file
`	stapp01 ansible_user=tony     ansible_ssh_pass=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'`
