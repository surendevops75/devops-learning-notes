# Ansible Templates and Jinja2

## Introduction

In real environments, configuration files differ between environments.

Examples:

```text
DEV

QA

PROD
```

Database hosts may be different.

---

# What is a Template?

A template is:

```text
Fixed Form

+

Placeholders
```

---

# Example

Template:

```text
DB_HOST={{ db_host }}
```

---

Runtime:

```yaml
db_host: mongodb-dev.daws86s.fun
```

---

Generated File:

```text
DB_HOST=mongodb-dev.daws86s.fun
```

---

# What is Templatization?

Templatization means:

```text
Keeping Placeholders

Providing Values At Runtime
```

---

# Why Templates?

Without Templates:

```text
dev.conf

qa.conf

prod.conf
```

Multiple files.

---

With Templates:

```text
Single Template

Dynamic Values
```

---

# Jinja2

Ansible templates use:

```text
Jinja2
```

templating engine.

---

# Jinja2 Syntax

```yaml
{{ variable }}
```

---

# Example

Template:

```text
DB_HOST={{ db_host }}
PORT={{ db_port }}
```

---

Variables:

```yaml
db_host: mongodb.daws86s.fun
db_port: 27017
```

---

Generated Output

```text
DB_HOST=mongodb.daws86s.fun
PORT=27017
```

---

# Template Location

Roles:

```text
roles/

└── catalogue

    └── templates
```

---

Example:

```text
mongo.repo.j2

catalogue.service.j2
```

---

# Template Module

Used to process Jinja2 files.

Example:

```yaml
- name: Copy Template

  template:
    src: catalogue.service.j2
    dest: /etc/systemd/system/catalogue.service
```

---

# What Happens Internally?

```text
Read Template
       │
       ▼
Replace Variables
       │
       ▼
Generate Final File
       │
       ▼
Copy To Target Server
```

---

# Real DevOps Example

Template:

```text
Environment=MONGO_URL={{ mongo_url }}
```

---

DEV:

```text
Environment=MONGO_URL=mongodb-dev.daws86s.fun
```

---

QA:

```text
Environment=MONGO_URL=mongodb-qa.daws86s.fun
```

---

PROD:

```text
Environment=MONGO_URL=mongodb.daws86s.fun
```

---

# Frequently Asked Interview Questions

## What Is a Template?

A template is a file containing placeholders that are replaced with actual values during execution.

---

## What Is Jinja2?

Jinja2 is the templating engine used by Ansible to generate dynamic configuration files.

---

## Why Use Templates?

Templates eliminate duplicate configuration files and support environment-specific values.

---

## Which Module Processes Templates?

```yaml
template
```

module.

---

# Key Takeaways

- Templates contain placeholders.
- Jinja2 is the templating engine used by Ansible.
- Variables are replaced at runtime.
- Templates support DEV, QA, and PROD environments.
- The template module generates final configuration files on target servers.