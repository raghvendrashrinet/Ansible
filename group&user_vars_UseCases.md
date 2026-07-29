# Real-World Production Use Cases of `group_vars` and `host_vars`

## Objective

Understand when to use **group_vars** and **host_vars** in real production environments.

---

# Rule of Thumb

| Use `group_vars` | Use `host_vars` |
|------------------|-----------------|
| Value is common for all hosts in a group | Value is unique to one host |
| Shared configuration | Host-specific exception |
| Default values | Overrides |

> **Best Practice:** Store common settings in `group_vars` and only exceptions in `host_vars`.

---

# Typical Project Structure

```text
inventory/

├── inventory.ini

├── group_vars/
│   ├── app_servers.yml
│   ├── db_servers.yml
│   └── load_balancers.yml

├── host_vars/
│   ├── stapp01.yml
│   ├── stapp02.yml
│   ├── stdb01.yml
│   └── lb01.yml

├── templates/

├── roles/

└── playbooks/
```

---

# Production Use Cases

## 1. Different Server IPs

### group_vars/app_servers.yml

```yaml
package_name: nginx
service_port: 80
```

### host_vars/stapp01.yml

```yaml
vip: 10.10.10.21
```

### host_vars/stapp02.yml

```yaml
vip: 10.10.10.22
```

**Use Case**

Each server has a unique IP or Virtual IP.

---

## 2. SSL Certificates

### host_vars/stapp01.yml

```yaml
ssl_cert: /etc/nginx/certs/app1.crt
ssl_key: /etc/nginx/certs/app1.key
```

### host_vars/stapp02.yml

```yaml
ssl_cert: /etc/nginx/certs/app2.crt
ssl_key: /etc/nginx/certs/app2.key
```

Jinja2

```jinja
ssl_certificate {{ ssl_cert }};
ssl_certificate_key {{ ssl_key }};
```

**Use Case**

Each application has its own SSL certificate.

---

## 3. Different DNS Names

```yaml
# host_vars/stapp01.yml
server_name: api.company.com
```

```yaml
# host_vars/stapp02.yml
server_name: auth.company.com
```

```yaml
# host_vars/stapp03.yml
server_name: payment.company.com
```

**Use Case**

Each server hosts a different website or API.

---

## 4. Different Memory / CPU Limits

```yaml
# host_vars/stapp01.yml
memory_limit: 8G
```

```yaml
# host_vars/stapp02.yml
memory_limit: 32G
```

Jinja2

```jinja
worker_rlimit_mem {{ memory_limit }};
```

**Use Case**

Servers have different hardware capacities.

---

## 5. Different Database Connections

```yaml
# host_vars/stapp01.yml
db_host: mysql-dev.company.com
```

```yaml
# host_vars/stapp02.yml
db_host: mysql-qa.company.com
```

```yaml
# host_vars/stapp03.yml
db_host: mysql-prod.company.com
```

**Use Case**

Applications connect to different databases.

---

## 6. Different Application Versions (Blue/Green Deployment)

### group_vars/app_servers.yml

```yaml
app_version: 3.4.2
```

### host_vars/stapp01.yml

```yaml
app_version: 3.5.0
```

**Use Case**

Deploy a new version to one server before rolling it out everywhere.

---

## 7. Maintenance Mode

### host_vars/stapp03.yml

```yaml
maintenance: true
```

Jinja2

```jinja
{% if maintenance %}
return 503;
{% endif %}
```

**Use Case**

Temporarily remove one server from production.

---

## 8. Load Balancer Weight

### host_vars/stapp01.yml

```yaml
weight: 50
```

### host_vars/stapp02.yml

```yaml
weight: 100
```

Jinja2

```jinja
server app01 {{ ansible_host }}:80 weight {{ weight }}
```

**Use Case**

Distribute traffic unevenly between servers.

---

## 9. Monitoring Labels

### host_vars/stapp01.yml

```yaml
environment: production
region: us-east
```

### host_vars/stapp02.yml

```yaml
environment: production
region: eu-west
```

**Use Case**

Monitoring systems (Prometheus, Grafana, Datadog) identify servers by region and environment.

---

## 10. Backup Schedule

### host_vars/stdb01.yml

```yaml
backup_time: "01:00"
```

### host_vars/stdb02.yml

```yaml
backup_time: "03:00"
```

Cron Template

```jinja
0 {{ backup_time }} * * * backup.sh
```

**Use Case**

Different database servers run backups at different times.

---

# What Typically Goes in group_vars?

```yaml
package_name: nginx
service_name: nginx
service_port: 80

document_root: /var/www/html
log_dir: /var/log/nginx

timezone: UTC
dns_server: 8.8.8.8
ntp_server: time.company.com

service_enabled: true
service_state: started
```

These values are common to every server in the group.

---

# What Typically Goes in host_vars?

```yaml
server_name: api.company.com

ssl_cert: api.crt
ssl_key: api.key

memory_limit: 32G

backup_time: "01:00"

weight: 100

maintenance: true

app_version: 3.5.0

vip: 10.10.10.21
```

These values apply to only one host.

---

# Example: Banking Environment

### group_vars/app_servers.yml

```yaml
package_name: nginx
service_name: nginx

service_port: 443

log_level: INFO

tls_version: TLS1.3
```

All 100 application servers inherit these values.

---

### host_vars/app45.yml

```yaml
server_name: payments.bank.com

ssl_cert: payments.crt
ssl_key: payments.key
```

---

### host_vars/app46.yml

```yaml
server_name: login.bank.com

ssl_cert: login.crt
ssl_key: login.key
```

The playbook remains exactly the same.

Only the data changes.

---

# Visual Flow

```text
                     Inventory
                         │
                         ▼
                 Select Target Hosts
                         │
                         ▼
                  group_vars Loaded
                         │
          (Common configuration)
                         │
                         ▼
                  host_vars Loaded
                  (Host overrides)
                         │
                         ▼
              Ansible Merges Variables
                         │
                         ▼
               Jinja2 Renders Templates
                         │
                         ▼
           Configuration Applied to Host
```

---

# Summary

| Scenario | group_vars | host_vars |
|----------|------------|-----------|
| Install package | ✅ | ❌ |
| Service name | ✅ | ❌ |
| Default application version | ✅ | ❌ |
| Override application version | ❌ | ✅ |
| SSL certificate | ❌ | ✅ |
| Server DNS name | ❌ | ✅ |
| Database hostname | ❌ | ✅ |
| Backup schedule | ❌ | ✅ |
| Maintenance mode | ❌ | ✅ |
| Load balancer weight | ❌ | ✅ |
| Memory/CPU limits | ❌ | ✅ |
| VIP / Static IP | ❌ | ✅ |

---

# Key Takeaways

- **Inventory** defines *which hosts* exist.
- **group_vars** defines *shared configuration* for a group.
- **host_vars** defines *host-specific overrides*.
- **Ansible merges all variables** before execution.
- **Jinja2 only renders templates** using the variables Ansible provides.
- In production, **90–95% of configuration usually lives in `group_vars`**, while **`host_vars` is reserved for exceptions**.
