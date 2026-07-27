

### Method 1 : Automated Provisioning via Cloud-Init & IaC (Cloud & VMs).
In cloud  or virtualized infra(VMware, OpenStack), target nodes are provisioned using IaC like Terraform. 
`Cloud-init` runs on initial boot to automatically inject the Ansible control node’s public SSH key and configure passwordless sudo access.

##### Step 1: Generate SSH Keypair on Control Node
`ssh-keygen -t ed25519 -C "ansible-control-node" -f ~/.ssh/ansible_id_ed25519 -N ""`
##### Step 2: Define Cloud-Init Configuration (cloud-init.yaml)
Create a cloud-init user-data configuration file,This script runs on the target node during first boot to:
1. Create a dedicated ansible user.
2. Inject the Ansible public key into ~ansible/.ssh/authorized_keys
3. Configure passwordless sudo for administrative privilege
```yaml
#cloud-config
users:
  - name: ansible
    gecos: Ansible Automation User
    groups: sudo, wheel
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... ansible-control-node

package_update: true
```
#####  Step 3: Deploy Target Infrastructure via Terraform
Pass the cloud-init configuration when provisioning target servers using Terraform (e.g., Azure VM):
```Terraform
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "rg-webserver"
  location = "East US"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-webserver"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = "subnet-webserver"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "nic" {
  name                = "nic-webserver"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "web-server-01"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  size                = "Standard_B1s"

  admin_username      = "azureuser"
  network_interface_ids = [azurerm_network_interface.nic.id]

  # Ubuntu 22.04 image
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  # Equivalent to AWS user_data
  custom_data = filebase64("cloud-init.yaml")

  tags = {
    Name = "web-server-01"
    Env  = "production"
  }
}
```
##### Step 4: Configure Ansible Inventory & Connection
Define your target nodes in your Ansible inventory (inventory.ini):
```Ini, TOML
[webservers]
192.168.1.50
192.168.1.51

[webservers:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/ansible_id_ed25519
ansible_python_interpreter=/usr/bin/python3
```
##### Step 5: Verify Connectivity
Run the Ansible ping 
```bash
ansible webservers -m ping -i inventory.ini
```

---
### Method 2: Accessing Private Subnets via Bastion Host (ProxyJump)
In enterprise networks, target servers reside in private subnets with no direct internet access or public IP addresses. To manage these, the Ansible Control Node routes its SSH traffic through a secure Bastion Host (Jump Box) located in a DMZ or public subnet using SSH ProxyJump.
##### Step 1: Understand the Network Architecture
- Control Node: Enterprise management server or CI/CD runner.
- Bastion Host (Jump Box): Public IP 203.0.113.10 (resolvable or reachable from Control Node).
- Target Nodes: Private IPs 10.0.2.50, 10.0.2.51 (unreachable directly from Control Node).
```
[ Ansible Control Node ] 
          │
          │ (Public SSH / Port 22)
          ▼
  [ Bastion Host ] (203.0.113.10)
          │
          │ (Private VPC / Subnet)
          ▼
[ Private Target Nodes ] (10.0.2.50, 10.0.2.51)
```
##### Step 2: Configure SSH ProxyJump Configuration
On the Ansible Control Node, configure ~/.ssh/config to automate routing through the Bastion Host:
```Ini, TOML
# SSH Config: ~/.ssh/config

# 1. Bastion Host Entry
Host bastion
    HostName 203.0.113.10
    User bastion-admin
    IdentityFile ~/.ssh/bastion_key.pem
    StrictHostKeyChecking no

# 2. Private Subnet Target Nodes (using ProxyJump),jumps to target through jump server
Host 10.0.2.*
    User ansible
    IdentityFile ~/.ssh/ansible_id_ed25519
    ProxyJump bastion
    StrictHostKeyChecking no
```
Set the correct file permissions on the config file:
```
chmod 600 ~/.ssh/config
```
##### Step 3: Configure Ansible Inventory
In your inventory.ini, reference the private IP addresses directly. Ansible will automatically utilize your ~/.ssh/config ProxyJump settings:
```Ini, TOML
[private_db_servers]
10.0.2.50
10.0.2.51

[private_db_servers:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/ansible_id_ed25519
```
##### Step 4: Alternative Method (Inline ansible_ssh_common_args)
If you do not want to modify ~/.ssh/config on the control node host, you can define the ProxyJump directly inside your Ansible inventory variables:
```Ini, TOML
[private_db_servers]
10.0.2.50
10.0.2.51

[private_db_servers:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/ansible_id_ed25519
ansible_ssh_common_args='-o ProxyJump=bastion-admin@203.0.113.10 -o IdentityFile=~/.ssh/bastion_key.pem'
```
##### Step 5: Verify Connectivity
`ansible private_db_servers -m ping -i inventory.ini`

---
### Method 3 : Short-Lived SSH Certificates & HashiCorp Vault (Zero Static Keys)
In high-security production environments, storing permanent SSH public/private key pairs on target servers introduces security risks,
enterprises use HashiCorp Vault as an SSH Certificate Authority (CA).
- Ansible requests a temporary, signed SSH certificate (valid for 5–15 minutes). Target nodes are configured to trust Vault’s CA, allowing seamless access without storing static keys on target servers.
##### Step 1: Configure Target Nodes to Trust Vault CA
On all target nodes, download Vault’s public SSH CA key and add it to the SSH configuration:
- 1. Save the Vault CA public key:
     ```
     sudo curl -s -o /etc/ssh/trusted-user-ca-keys.pem https://vault.internal.domain/v1/ssh-client-signer/public_key
     ```
- 2. Update /etc/ssh/sshd_config on target nodes:
  ```Ini, TOML
  TrustedUserCAKeys /etc/ssh/trusted-user-ca-keys.pem
  ```
- 3. Restart the SSH service:
  ```
  sudo systemctl restart sshd
  ```
  ##### Step 2: Request a Short-Lived SSH Certificate on the Control Node
  Before running playbooks, the Ansible Control Node authenticates with HashiCorp Vault and requests a signed certificate using the Vault CLI or a script:
  - 1. Generate a temporary keypair on Control Node:
    ```
    ssh-keygen -t ed25519 -f ~/.ssh/ansible_temp_key -N ""
    ```
  - 2. Request Vault to sign the public key (valid for 10 minutes):
    ```
    vault write -field=signed_key ssh-client-signer/sign/ansible-role \
    public_key=@~/.ssh/ansible_temp_key.pub \
    valid_principals="ansible" \
    ttl="10m" > ~/.ssh/ansible_temp_key-cert.pub
    ```
  ##### Step 3: Configure Ansible Inventory
Configure Ansible to use both the temporary private key and its corresponding signed certificate:
```Ini, TOML
[all:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/ansible_temp_key
ansible_ssh_common_args='-o CertificateFile=~/.ssh/ansible_temp_key-cert.pub'
```
##### Step 4: Automate Certificate Generation via Ansible Plugin / Lookup
To avoid manually requesting certificates, you can use HashiCorp Vault lookup plugins directly in your Ansible playbooks or execution scripts:
```yaml
---
- name: Verify Vault SSH Certificate Connectivity
  hosts: all
  gather_facts: no
  tasks:
    - name: Ping target nodes
      ansible.builtin.ping:
```
Execution script wrapper (`run-ansible.sh`):
```bash
#!/usr/bin/env bash
# 1. Authenticate & fetch temporary SSH certificate
vault write -field=signed_key ssh-client-signer/sign/ansible-role \
    public_key=@~/.ssh/ansible_temp_key.pub \
    valid_principals="ansible" \
    ttl="10m" > ~/.ssh/ansible_temp_key-cert.pub

# 2. Execute playbook
ansible-playbook -i inventory.ini site.yml
```
##### Step 5: Verify Connectivity
`ansible all -m ping -i inventory.ini`

---
### Method 4: Enterprise Orchestration Platforms (Ansible Automation Platform / AWX)
In enterprise environments, executing playbooks manually from a local control node command line is often replaced by centralized orchestration tools like Red Hat Ansible Automation Platform (AAP) or AWX (open-source upstream).

Instead of engineers holding SSH keys or passwords on their local machines, AAP/AWX securely stores machine credentials in an encrypted database (or syncs them with enterprise secret managers like CyberArk or HashiCorp Vault) and manages SSH connectivity automatically during job execution.

##### Step 1: Store Credentials in AAP / AWX Vault
In the AWX/AAP Web UI, administrative users create Machine Credentials that are decoupled from individual user access:
1. Navigate to Credentials > Add Credential.

2. Set Credential Type to Machine.

3. Provide authentication details:
  - Username: ansible
  - SSH Private Key: Paste the enterprise automation SSH private key (or configure HashiCorp Vault / CyberArk lookup).
  - Privilege Escalation Method: sudo
  - Sudo Username: root
AAP/AWX encrypts these keys at rest and injects them only into ephemeral execution environment containers when jobs run

##### Step 2: Define Dynamic or Static Inventories
Target hosts are managed centrally inside AAP/AWX rather than in flat text files:
1. Navigate to Inventories > Add Inventory.

2. Add target hosts or configure Inventory Sources (e.g., AWS EC2, VMware vCenter, or Azure dynamic inventory plugins) to auto-discover target nodes.

##### Step 3: Create Job Templates & Attach Credentials
Create a Job Template to tie together your playbook, inventory, and machine credentials
1. Go to Templates > Add Job Template.

2. Select your Inventory (e.g., Production-Web-Servers).

3. Select your Project (Git repository containing your Ansible playbooks).

4. Select your Playbook (e.g., deploy_app.yml).

5. Under Credentials, attach the Machine Credential created in Step 1.
##### Step 4: Execute Playbooks via Web UI, API, or Webhooks
Once configured, playbooks can be triggered without any SSH setup on developer laptops:
  - Manual Execution: Click the Launch button in the AWX dashboard.
  - REST API: Trigger execution programmatically from external systems:
```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your_awx_api_token>" \
  https://awx.internal.domain/api/v2/job_templates/12/launch/
```
- GitOps / Webhooks: Automatically trigger execution whenever changes are pushed to your GitHub/GitLab repository.
##### Step 5: Verification & Audit Logging
- AAP/AWX launches an isolated container (Execution Environment) to execute the job.
- SSH keys are dynamically passed to the runner container for the duration of the job run and destroyed immediately after completion.
- Full stdout logs, task timing, and detailed audit trails are captured in the AWX database for compliance reporting.  
---
### Method 5: Windows Targets (WinRM / OpenSSH)
- Use Case: Managing Windows Server infrastructure.

- Flow: Control node uses pywinrm over HTTPS (port 5986) or OpenSSH.

Key Inventory:

```Ini, TOML
[win_servers:vars]
ansible_user=Administrator
ansible_connection=winrm
ansible_winrm_transport=credssp
```
