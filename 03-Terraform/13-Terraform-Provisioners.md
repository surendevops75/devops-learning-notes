# Terraform Provisioners

## Introduction

Terraform is primarily an:

```text
Infrastructure Provisioning Tool
```

Its responsibility is:

```text
Create Infrastructure

Update Infrastructure

Delete Infrastructure
```

Examples:

```text
EC2 Instances

VPC

Security Groups

Route53 Records

Load Balancers
```

---

# Infrastructure vs Configuration

Terraform is excellent at:

```text
Infrastructure Provisioning
```

Examples:

```text
Create EC2

Create VPC

Create Security Groups
```

However, applications often require:

```text
Install Packages

Configure Services

Deploy Application Code

Start Services
```

This is:

```text
Configuration Management
```

Typically handled by:

```text
Ansible

Chef

Puppet
```

---

# Why Do Provisioners Exist?

Sometimes after creating infrastructure, we want to execute commands.

Example:

```text
Create EC2 Instance
        ↓
Install Nginx
        ↓
Start Nginx Service
```

Terraform provides:

```text
Provisioners
```

for such situations.

---

# What is a Provisioner?

A Provisioner executes:

```text
Scripts

Commands

Programs
```

during resource creation or destruction.

---

# Important Note

HashiCorp recommends:

```text
Avoid Provisioners When Possible
```

Provisioners should be considered:

```text
Last Resort
```

because they are harder to manage and troubleshoot than declarative infrastructure.

---

# Types of Provisioners

Terraform commonly provides:

```text
local-exec

remote-exec
```

---

# local-exec Provisioner

Runs commands on:

```text
Machine Running Terraform
```

Not on the EC2 instance.

---

# Architecture

```text
Terraform Server
        ↓
local-exec
        ↓
Command Executes Locally
```

---

# Syntax

```hcl
provisioner "local-exec" {

  command = "command_to_execute"

}
```

---

# Example

```hcl
resource "aws_instance" "frontend" {

  ami = "ami-xxxxxxxx"

  instance_type = "t3.micro"

  provisioner "local-exec" {

    command = "echo Instance Created"

  }

}
```

---

# Execution Flow

```text
EC2 Created
      ↓
local-exec Runs
      ↓
Output Displayed
```

---

# Production Example

Store instance IP in a file.

```hcl
provisioner "local-exec" {

  command = "echo ${self.private_ip} >> inventory"

}
```

---

Result:

```text
inventory
```

contains:

```text
10.0.1.10
```

---

# What is self?

Inside a resource:

```hcl
self
```

refers to:

```text
Current Resource
```

Example:

```hcl
self.private_ip

self.public_ip

self.id
```

---

# remote-exec Provisioner

Runs commands on:

```text
Remote Server
```

after it is created.

---

# Architecture

```text
Terraform
      ↓
SSH Connection
      ↓
Remote Server
      ↓
Execute Commands
```

---

# Syntax

```hcl
provisioner "remote-exec" {

  inline = [

    "command1",

    "command2"

  ]

}
```

---

# Connection Block

Terraform requires connection details.

Example:

```hcl
connection {

  type = "ssh"

  user = "ec2-user"

  password = "password"

  host = self.public_ip

}
```

---

# Complete Example

```hcl
resource "aws_instance" "frontend" {

  ami = "ami-xxxxxxxx"

  instance_type = "t3.micro"

  connection {

    type = "ssh"

    user = "ec2-user"

    password = "DevOps321"

    host = self.public_ip

  }

  provisioner "remote-exec" {

    inline = [

      "sudo dnf install nginx -y",

      "sudo systemctl start nginx"

    ]

  }

}
```

---

# Execution Flow

```text
Create EC2
      ↓
SSH Login
      ↓
Install Nginx
      ↓
Start Nginx
```

---

# Running Shell Scripts

Instead of inline commands:

```hcl
provisioner "remote-exec" {

  script = "mongodb.sh"

}
```

Terraform uploads and executes:

```text
mongodb.sh
```

on the remote machine.

---

# Multiple Commands

Example:

```hcl
provisioner "remote-exec" {

  inline = [

    "sudo dnf update -y",

    "sudo dnf install nginx -y",

    "sudo systemctl enable nginx",

    "sudo systemctl start nginx"

  ]

}
```

---

# File Provisioner

Terraform can copy files to remote servers.

Example:

```hcl
provisioner "file" {

  source = "nginx.conf"

  destination = "/tmp/nginx.conf"

}
```

---

# Execution Order

Example:

```text
Create EC2
      ↓
Copy File
      ↓
Execute Commands
```

---

# Calling Ansible from Terraform

One of the most common real-world use cases.

Terraform:

```text
Creates Infrastructure
```

Ansible:

```text
Configures Infrastructure
```

---

# Example Workflow

```text
Terraform
      ↓
Create EC2
      ↓
Update Inventory
      ↓
Run Ansible Playbook
```

---

# Example

```hcl
provisioner "local-exec" {

  command = "ansible-playbook -i inventory playbook.yaml"

}
```

---

# Why Use This?

Terraform handles:

```text
Infrastructure Provisioning
```

Ansible handles:

```text
Configuration Management
```

This follows industry best practices.

---

# Real Production Example

Roboshop Deployment:

```text
Terraform
      ↓
Create MongoDB Server
      ↓
Create Redis Server
      ↓
Create Catalogue Server
      ↓
Update Inventory
      ↓
Run Ansible
      ↓
Configure Services
```

---

# When Provisioners Run

Default:

```text
Resource Creation
```

---

# Destroy Provisioner

Example:

```hcl
provisioner "local-exec" {

  when = destroy

  command = "echo Resource Deleted"

}
```

---

Runs during:

```bash
terraform destroy
```

---

# Why Provisioners Are Considered Last Resort

Problems:

```text
Not Idempotent

Hard To Debug

Can Fail Midway

Not Tracked In State
```

---

# Example Problem

Terraform creates:

```text
EC2 Instance
```

but:

```text
remote-exec
```

fails.

Result:

```text
Infrastructure Exists

Configuration Incomplete
```

---

# Better Alternative

Preferred Approach:

```text
Terraform
      ↓
Create Infrastructure
      ↓
Ansible
      ↓
Configure Infrastructure
```

Instead of:

```text
Terraform
      ↓
Terraform Provisioners
```

---

# Common Mistakes

## Installing Applications with Provisioners

Bad for large-scale environments.

Use:

```text
Ansible
```

instead.

---

## Hardcoding Passwords

Bad:

```hcl
password = "DevOps321"
```

Use:

```text
SSH Keys

Secrets Manager

Vault
```

---

## Running Large Scripts

Avoid huge provisioning scripts.

---

## Using Provisioners for Everything

Terraform should focus on:

```text
Infrastructure
```

not complete application deployment.

---

# Best Practices

## Use Provisioners Sparingly

---

## Prefer Configuration Management Tools

Examples:

```text
Ansible

Chef

Puppet
```

---

## Use local-exec for Small Tasks

Examples:

```text
Generate Inventory

Call APIs

Trigger Automation
```

---

## Avoid Business Logic in Provisioners

---

## Use Secure Authentication

Avoid plain-text passwords.

---

# Frequently Asked Interview Questions

## What is a Terraform Provisioner?

### Short Answer

A Provisioner executes commands or scripts during resource creation or destruction.

### Detailed Explanation

Provisioners allow Terraform to perform post-deployment actions such as running commands, copying files, or triggering automation workflows. They are useful for bootstrapping but should be used carefully.

### Production Example

Generating an Ansible inventory file after creating EC2 instances.

### Follow-Up Questions

- What types of provisioners exist?
- Why are provisioners considered a last resort?

---

## What is local-exec?

### Short Answer

local-exec runs commands on the machine where Terraform is executed.

### Detailed Explanation

The command executes locally and can interact with files, APIs, scripts, and external tools.

### Production Example

Running an Ansible playbook after infrastructure creation.

### Follow-Up Questions

- Does local-exec run on EC2?
- What are common local-exec use cases?

---

## What is remote-exec?

### Short Answer

remote-exec runs commands on a remote server after Terraform creates it.

### Detailed Explanation

Terraform connects using SSH or WinRM and executes commands directly on the target machine.

### Production Example

Installing Nginx on a newly created EC2 instance.

### Follow-Up Questions

- What is a connection block?
- How does Terraform connect to the server?

---

## Why Are Provisioners Considered Last Resort?

### Short Answer

Because they are harder to maintain, debug, and manage than declarative infrastructure.

### Detailed Explanation

Provisioners introduce imperative behavior into Terraform, making deployments less predictable and harder to recover from when failures occur.

### Production Example

A remote-exec script fails after EC2 creation, leaving infrastructure partially configured.

### Follow-Up Questions

- What alternatives exist?
- Why is Ansible preferred?

---

## How Is Terraform Commonly Integrated with Ansible?

### Short Answer

Terraform creates infrastructure and Ansible configures it.

### Detailed Explanation

Terraform focuses on infrastructure provisioning, while Ansible handles package installation, application deployment, and server configuration.

### Production Example

Terraform creates Roboshop servers and then triggers Ansible playbooks to configure MongoDB, Redis, Catalogue, and other services.

### Follow-Up Questions

- Why separate Terraform and Ansible?
- How do you pass inventory information to Ansible?

---

# Key Takeaways

- Provisioners execute commands during resource lifecycle events.
- local-exec runs locally.
- remote-exec runs on remote servers.
- file provisioners copy files to remote systems.
- Terraform and Ansible are commonly used together.
- Provisioners should be used sparingly.
- Infrastructure provisioning and configuration management should ideally remain separate.
- Provisioners are a common Terraform interview topic.