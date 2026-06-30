# Ansible Handlers and Notifications

## Introduction

Some actions should happen only when changes occur.

Example:

```text
Configuration Changed
        │
        ▼
Restart Service
```

If nothing changed:

```text
No Restart Required
```

---

# Problem

Suppose:

```yaml
Copy nginx.conf
```

and then:

```yaml
Restart nginx
```

every playbook execution.

This causes:

```text
Unnecessary Restarts

Service Disruptions

Longer Execution Time
```

---

# Solution

Use:

```text
Handlers
```

---

# What is a Handler?

A handler is a special task that executes only when notified.

---

# Workflow

```text
Task Changes Something
        │
        ▼
Notify Handler
        │
        ▼
Handler Executes
```

---

# Example

Task:

```yaml
- name: Copy Config

  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

  notify:
    - Restart Nginx
```

---

Handler:

```yaml
- name: Restart Nginx

  service:
    name: nginx
    state: restarted
```

---

# Result

Configuration Changed:

```text
Handler Executes
```

---

No Change:

```text
Handler Skipped
```

---

# Benefits

```text
Efficient

Faster

Avoids Unnecessary Restarts
```

---

# Frequently Asked Interview Questions

## What Is a Handler?

A handler is a task that executes only when notified by another task.

---

## Why Are Handlers Important?

Handlers prevent unnecessary service restarts and improve automation efficiency.

---

# Key Takeaways

- Handlers execute only when notified.
- Commonly used for service restarts.
- Improve performance and reduce disruptions.