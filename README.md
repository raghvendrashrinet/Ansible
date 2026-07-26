## Installing Ansible

#### Option 1: Install via pip (Recommended)
-*Step 1*: Install Python & pip
  Ensure Python 3 (3.9+) and pip are installed on your machine
```bash
 sudo apt update
 sudo apt install -y python3 python3-pip python3-venv
```
- *Step 2*: Create and Activate a Virtual Environment
```bash
 python3 -m venv ~/ansible-venv
 source ~/ansible-venv/bin/activate
```
- *Step 3*: Install Ansible
```bash
 pip install --upgrade pip
 pip install ansible
```
Verify
```
 ansible --version
```

#### Option 2: Install via OS Package Manager
```
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
```
Setup and intialize the  directory 
```
cd Ansible

# Create an initial Ansible structure
mkdir -p inventory group_vars host_vars roles

# Create a basic configuration file and inventory
touch ansible.cfg inventory/hosts site.yml
```
---

#### Basic ansible.cfg Example
Create a minimal configuration file (ansible.cfg) in your repo root:

```Ini, TOML
[defaults]
inventory = ./inventory/hosts
host_key_checking = False
```
#### Basic inventory/hosts Example
Add your target servers to inventory/hosts:

```Ini, TOML
[webservers]
# Replace with target server IP or hostname
192.168.1.50 ansible_user=ubuntu
```
Test Connection (Ping): Run the built-in ping module to verify that your control node can connect to your target host:
`ansible webservers -m ping`


