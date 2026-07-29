# Jinja2 in Production - Real World Use Cases

## Objective

Understand where **Jinja2** is used in real production environments.

> **Remember**
>
> - **Ansible** collects variables.
> - **Jinja2** renders templates.
> - **Target Server** receives the final configuration file.

---

# Where Jinja2 Fits

```
Inventory
      │
      ▼
group_vars
      │
      ▼
host_vars
      │
      ▼
Facts
      │
      ▼
Ansible
(Merges Variables)
      │
      ▼
Jinja2
(Renders Template)
      │
      ▼
Configuration File
      │
      ▼
Target Server
```

---

# 1. Nginx Configuration

## Template

```jinja
server {

    listen {{ service_port }};

    server_name {{ server_name }};

    root {{ document_root }};

}
```

## Variables

```yaml
service_port: 80
server_name: api.company.com
document_root: /var/www/html
```

## Generated File

```nginx
server {

    listen 80;

    server_name api.company.com;

    root /var/www/html;

}
```

### Production Use

One template can configure hundreds of Nginx servers.

---

# 2. Apache Virtual Hosts

## Template

```jinja
<VirtualHost *:80>

ServerName {{ server_name }}

DocumentRoot {{ document_root }}

</VirtualHost>
```

## Generated Output

```apache
<VirtualHost *:80>

ServerName shop.company.com

DocumentRoot /var/www/shop

</VirtualHost>
```

### Production Use

Automatically create Virtual Hosts for many websites.

---

# 3. HAProxy Configuration

## Template

```jinja
backend app_servers

{% for server in groups['app_servers'] %}

server {{ server }}
{{ hostvars[server].ansible_host }}:80

{% endfor %}
```

## Generated Output

```text
backend app_servers

server stapp01 192.168.1.10:80

server stapp02 192.168.1.11:80

server stapp03 192.168.1.12:80
```

### Production Use

Automatically generate backend servers from inventory.

---

# 4. Kubernetes Deployment YAML

## Template

```jinja
apiVersion: apps/v1

kind: Deployment

spec:

  replicas: {{ replicas }}

  template:

    spec:

      containers:

      - name: app

        image: {{ image_name }}:{{ image_version }}
```

## Variables

```yaml
replicas: 3

image_name: nginx

image_version: 1.28
```

## Generated YAML

```yaml
replicas: 3

image: nginx:1.28
```

### Production Use

Deploy different versions to Dev, QA and Production.

---

# 5. Docker Compose

## Template

```jinja
services:

  app:

    image: {{ image }}

    ports:

      - "{{ port }}:80"
```

Variables

```yaml
image: nginx

port: 8080
```

Generated

```yaml
services:

  app:

    image: nginx

    ports:

      - "8080:80"
```

### Production Use

Generate Docker Compose files for multiple environments.

---

# 6. Application Configuration

Example

```
application.properties
```

## Template

```jinja
server.port={{ app_port }}

db.host={{ db_host }}

db.user={{ db_user }}

db.password={{ db_password }}
```

Variables

```yaml
app_port: 8080

db_host: mysql.company.com

db_user: appuser

db_password: Password123
```

Generated

```properties
server.port=8080

db.host=mysql.company.com

db.user=appuser

db.password=Password123
```

### Production Use

Every environment gets different database settings.

---

# 7. Systemd Service Files

## Template

```jinja
[Service]

User={{ app_user }}

WorkingDirectory={{ app_home }}

ExecStart={{ start_command }}
```

Generated

```ini
[Service]

User=tomcat

WorkingDirectory=/opt/tomcat

ExecStart=/opt/tomcat/bin/start.sh
```

### Production Use

Deploy Java applications with different startup commands.

---

# 8. Cron Jobs

## Template

```jinja
{{ backup_time }} root /opt/scripts/backup.sh
```

Variables

```yaml
backup_time: "0 2 * * *"
```

Generated

```
0 2 * * * root /opt/scripts/backup.sh
```

### Production Use

Each server runs backups at different times.

---

# 9. Monitoring Configuration

Prometheus

## Template

```jinja
global:

  scrape_interval: {{ scrape_interval }}
```

Variables

```yaml
scrape_interval: 15s
```

Generated

```yaml
global:

  scrape_interval: 15s
```

### Production Use

Generate monitoring configuration for different environments.

---

# 10. SSH Configuration

## Template

```jinja
Host {{ inventory_hostname }}

HostName {{ ansible_host }}

User {{ ansible_user }}
```

Generated

```text
Host stapp01

HostName 192.168.1.10

User tony
```

### Production Use

Automatically generate SSH client configuration.

---

# 11. Environment Files (.env)

## Template

```jinja
APP_NAME={{ app_name }}

APP_PORT={{ app_port }}

LOG_LEVEL={{ log_level }}
```

Variables

```yaml
app_name: inventory

app_port: 8080

log_level: INFO
```

Generated

```text
APP_NAME=inventory

APP_PORT=8080

LOG_LEVEL=INFO
```

### Production Use

Create environment-specific configuration files.

---

# 12. Firewall Rules

## Template

```jinja
allow {{ service_port }}/tcp
```

Variables

```yaml
service_port: 443
```

Generated

```
allow 443/tcp
```

### Production Use

Open different ports for different servers.

---

# 13. Kubernetes ConfigMap

## Template

```jinja
data:

  LOG_LEVEL: "{{ log_level }}"

  API_URL: "{{ api_url }}"
```

Variables

```yaml
log_level: INFO

api_url: https://api.company.com
```

Generated

```yaml
data:

  LOG_LEVEL: INFO

  API_URL: https://api.company.com
```

### Production Use

Different configuration for Dev, QA and Production.

---

# 14. Database Configuration

## Template

```jinja
bind-address={{ db_ip }}

max_connections={{ max_connections }}
```

Variables

```yaml
db_ip: 10.10.1.20

max_connections: 500
```

Generated

```ini
bind-address=10.10.1.20

max_connections=500
```

### Production Use

Tune database configuration per environment.

---

# 15. HTML Status Page

## Template

```jinja
<h1>{{ application_name }}</h1>

<p>Version {{ version }}</p>

<p>Environment {{ environment }}</p>
```

Variables

```yaml
application_name: Inventory Service

version: 2.5.1

environment: Production
```

Generated

```html
<h1>Inventory Service</h1>

<p>Version 2.5.1</p>

<p>Environment Production</p>
```

### Production Use

Display deployment information on internal dashboards.

---

# Most Common Jinja2 Features Used

## Variables

```jinja
{{ server_name }}
```

---

## Conditions

```jinja
{% if ssl_enabled %}

listen 443 ssl;

{% endif %}
```

---

## Loops

```jinja
{% for server in groups['app_servers'] %}

server {{ server }}

{% endfor %}
```

---

## Filters

```jinja
{{ server_name | upper }}

{{ filename | lower }}

{{ users | length }}

{{ ip_list | join(',') }}
```

---

## Access Dictionary Values

```jinja
{{ employee.name }}

{{ employee.department }}

{{ hostvars['stapp01'].ansible_host }}
```

---

# Production Workflow

```
Developer
      │
      ▼
Inventory
      │
      ▼
group_vars
      │
      ▼
host_vars
      │
      ▼
Facts
      │
      ▼
Playbook
      │
      ▼
Jinja2 Template
      │
      ▼
Generated Configuration
      │
      ▼
Copied to Target Server
      │
      ▼
Service Restarted
```

---

# Common Files Generated by Jinja2

| Configuration | Example |
|--------------|---------|
| Nginx | nginx.conf |
| Apache | httpd.conf |
| HAProxy | haproxy.cfg |
| Keepalived | keepalived.conf |
| Docker Compose | docker-compose.yml |
| Kubernetes | Deployment, Service, ConfigMap |
| Prometheus | prometheus.yml |
| Grafana | datasource.yml |
| MySQL | my.cnf |
| PostgreSQL | postgresql.conf |
| Redis | redis.conf |
| SSH | ssh_config |
| Cron | crontab |
| Systemd | *.service |
| Environment | .env |
| Application | application.properties |

---

# Key Takeaways

- Jinja2 **does not decide values**; it only renders templates.
- One template can generate hundreds or thousands of configuration files.
- It eliminates duplicate configuration files and follows the **DRY (Don't Repeat Yourself)** principle.
- Jinja2 is most commonly used to generate configuration files, Kubernetes manifests, Docker Compose files, service definitions, monitoring configurations, and application settings.
- The same template can produce different outputs simply by changing the variables supplied by Ansible.
