# Jinja2 Templating in Ansible - Understanding the Problem First

---

# Objective

Before learning Jinja2 syntax, understand **why Jinja2 exists** and **how Ansible uses it**.

> **Core Idea**
>
> Jinja2 was created to avoid writing and maintaining hundreds or thousands of almost identical configuration files.

---

# Real World Scenario

Imagine you have joined a company as a DevOps Engineer.

Your company manages **500 Linux web servers**.

```
web01
web02
web03
...
web500
```

Each server requires a configuration file.

Example:

For **web01**

```ini
hostname=web01
ip=10.0.1.10
environment=dev
port=8080
```

For **web02**

```ini
hostname=web02
ip=10.0.1.11
environment=qa
port=8080
```

For **web03**

```ini
hostname=web03
ip=10.0.1.12
environment=prod
port=80
```

---

# Problem

Observe these files carefully, Almost everything is identical.
Only these values change:
- hostname
- IP Address
- environment
- port

Without templates, you would have

```
config-web01.conf
config-web02.conf
config-web03.conf
...
config-web500.conf
```

Now imagine the company decides to change:

```
log_directory=/var/log/myapp
```

You now have to edit **500 files**.

This is:

- Time consuming
- Error prone
- Difficult to maintain

---

# Solution

Instead of creating 500 configuration files...

Create **one template**.

```
config.conf.j2
```

Contents

```jinja
hostname={{ hostname }}
ip={{ ansible_host }}
environment={{ env }}
port={{ port }}
```

Notice the placeholders.

```
{{ hostname }}
{{ ansible_host }}
{{ env }}
{{ port }}
```

These are **variables**.

---

# Question

Where do these variables come from?

Does Jinja2 know them?

**No.**

Ansible provides them.

---

# Step 1 - Inventory

Example inventory

```yaml
webservers:
  hosts:

    web01:
      ansible_host: 10.0.1.10

    web02:
      ansible_host: 10.0.1.11

    web03:
      ansible_host: 10.0.1.12
```

From this inventory Ansible learns

For web01

```
ansible_host = 10.0.1.10
```

For web02

```
ansible_host = 10.0.1.11
```

For web03

```
ansible_host = 10.0.1.12
```

---

# Step 2 - Variables

Suppose you also have

```yaml
env: production
port: 80
```

Now Ansible knows

```
env = production
port = 80
```

---

# Step 3 - Playbook

```yaml
- hosts: webservers

  tasks:

    - name: Create Configuration File

      template:
        src: config.conf.j2
        dest: /etc/myapp/config.conf
```

Notice something.

The playbook never says

```
hostname=web01
```

So...

How does Jinja2 know?

---

# What Actually Happens Internally

When Ansible starts working on **web01**

it collects every variable available.

Think of it as creating a dictionary.

```
{
    hostname      : web01,
    ansible_host  : 10.0.1.10,
    env           : production,
    port          : 80
}
```

This dictionary is given to Jinja2.

---

# Think of It Like This

```
                 Ansible

          Collect Variables

                    │

                    ▼

        +------------------------+
        | hostname = web01       |
        | ansible_host = 10.0.1.10 |
        | env = production       |
        | port = 80              |
        +------------------------+

                    │

                    ▼

             Jinja2 Template

hostname={{ hostname }}
ip={{ ansible_host }}
environment={{ env }}
port={{ port }}

                    │

                    ▼

             Generated File
```

---

# How Jinja2 Thinks

Jinja2 is extremely simple.

It asks

```
I need hostname.
```

Ansible replies

```
web01
```

Jinja2 replaces it.

Then Jinja2 asks

```
I need ansible_host.
```

Ansible replies

```
10.0.1.10
```

Jinja2 replaces it.

Then

```
env
```

Answer

```
production
```

Then

```
port
```

Answer

```
80
```

Finished.

---

# Generated File

```ini
hostname=web01
ip=10.0.1.10
environment=production
port=80
```

---

# Next Host

Ansible now starts processing **web02**.

Again it creates a variable dictionary.

```
{
    hostname      : web02,
    ansible_host  : 10.0.1.11,
    env           : production,
    port          : 80
}
```

Jinja2 receives exactly the same template.

```
hostname={{ hostname }}
ip={{ ansible_host }}
environment={{ env }}
port={{ port }}
```

But different variables.

Generated output

```ini
hostname=web02
ip=10.0.1.11
environment=production
port=80
```

---

# Next Host

Again...

Ansible processes **web03**

Dictionary

```
{
    hostname      : web03,
    ansible_host  : 10.0.1.12,
    env           : production,
    port          : 80
}
```

Generated file

```ini
hostname=web03
ip=10.0.1.12
environment=production
port=80
```

---

# Notice the Pattern

The template never changed.

```
hostname={{ hostname }}
ip={{ ansible_host }}
environment={{ env }}
port={{ port }}
```

Only the variables changed.

Different variables

↓

Different output

---

# Important Responsibilities

## Inventory

Knows

- Which servers exist
- Their IP addresses

---

## Variables

Store

- Environment
- Port
- Host specific settings

---

## Ansible

Responsible for

- Reading inventory
- Reading variables
- Gathering facts
- Building one dictionary for the current host
- Calling Jinja2

---

## Jinja2

Has only one responsibility.

Replace placeholders with values.

Nothing more.

Jinja2 does NOT know

- Inventory
- group_vars
- host_vars
- Playbooks
- Facts
- SSH
- Servers

It only receives variables.

---

# Internal Workflow

```
Inventory
      │
      ▼
Variables
      │
      ▼
Facts
      │
      ▼
Ansible
(Builds Variable Dictionary)
      │
      ▼
Jinja2 Template Engine
      │
      ▼
Rendered Configuration File
      │
      ▼
Copied to Target Server
```

---

# One Sentence Summary

**Ansible decides the values.**

**Jinja2 simply fills the blanks.**

---

# Mental Model

Think of Ansible as a manager.

Think of Jinja2 as a printer.

```
            Inventory
                 │
                 ▼
          Ansible Manager
                 │
        Collect Variables
                 │
                 ▼
         Jinja2 Printer
                 │
                 ▼
      Final Configuration File
```

The printer never decides what to print.

It only prints whatever the manager provides.

Jinja2 works exactly the same way.

---

# Key Takeaways

- Jinja2 is a template engine.
- It does not create data.
- It only replaces placeholders.
- Ansible provides all variables.
- One template can generate thousands of configuration files.
- This makes infrastructure automation scalable and easy to maintain.
