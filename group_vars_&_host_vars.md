# Understanding `group_vars` and `host_vars` in Ansible

## Objective

Learn where Ansible gets variables before passing them to Jinja2.

> **Remember**
>
> **Ansible collects variables → Jinja2 replaces placeholders.**

---

# Example Infrastructure

```
web01
web02
web03

db01
db02
db03
```

Grouped by role:

```
Web Servers
------------
web01
web02
web03

Database Servers
----------------
db01
db02
db03
```

---

# Inventory

```yaml
all:
  children:

    webservers:
      hosts:
        web01:
          ansible_host: 10.0.1.10
        web02:
          ansible_host: 10.0.1.11
        web03:
          ansible_host: 10.0.1.12

    dbservers:
      hosts:
        db01:
          ansible_host: 10.0.2.10
        db02:
          ansible_host: 10.0.2.11
        db03:
          ansible_host: 10.0.2.12
```

Inventory tells Ansible:

- Which hosts exist
- Their IP addresses
- Which group they belong to

---

# What is `group_vars`?

`group_vars` contains variables shared by **every host in a group**.

Example:

```
group_vars/
└── webservers.yml
```
*Note: file name 'webservers.yml` should be same as group name*

```yaml
port: 80
service_name: nginx
environment: production
```

All web servers automatically receive:

```
port = 80
service_name = nginx
environment = production
```

Similarly:

```
group_vars/
└── dbservers.yml
```

```yaml
port: 3306
service_name: mysql
environment: production
```

Every database server receives these values.

### Think of it as

> **Department policy**
>
> Every member follows the same rules.

---

# What is `host_vars`?

`host_vars` contains variables for **one specific host**.

Example:

```
host_vars/
└── web01.yml
```

```yaml
port: 8080
```

Result:

| Host | Port |
|------|------|
| web01 | 8080 |
| web02 | 80 |
| web03 | 80 |

Only **web01** is affected.

### Think of it as

> **Server-specific exception**

---

# How Ansible Builds Variables

For **web01**

### Step 1 – Inventory

```
hostname = web01
ansible_host = 10.0.1.10
```

### Step 2 – group_vars

```
port = 80
service_name = nginx
environment = production
```

### Step 3 – host_vars

```
port = 8080
```

Since `host_vars` is more specific, it overrides `group_vars`.

Final variables:

```
hostname = web01
ansible_host = 10.0.1.10
service_name = nginx
environment = production
port = 8080
```

These variables are passed to Jinja2.

---

# Visual Workflow

```
                Inventory
                     │
                     ▼
             group_vars
          (Shared variables)
                     │
                     ▼
              host_vars
       (Host-specific values)
                     │
                     ▼
         Ansible merges variables
                     │
                     ▼
                Jinja2
                     │
                     ▼
      Generated Configuration File
```

---

# Directory Structure

```
inventory/

├── hosts.yml

├── group_vars/
│   ├── webservers.yml
│   └── dbservers.yml

├── host_vars/
│   ├── web01.yml
│   └── db02.yml

├── templates/
│   └── nginx.conf.j2

└── playbook.yml
```

---

# Quick Comparison

| Feature | `group_vars` | `host_vars` |
|----------|--------------|-------------|
| Scope | Entire group | Single host |
| Purpose | Shared configuration | Host-specific configuration |
| Example | Port 80 for all web servers | Port 8080 only for web01 |
| Analogy | Department policy | Individual exception |

---

# Key Takeaways

- **Inventory** defines hosts and groups.
- **group_vars** applies variables to every host in a group.
- **host_vars** applies variables to one specific host.
- If the same variable exists in both, **host_vars overrides group_vars**.
- Ansible merges all variables and passes them to **Jinja2**.
- Jinja2 only replaces placeholders; it never reads inventory or variable files itself.
