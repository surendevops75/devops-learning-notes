# Terraform Provisioners and terraform_data

## Introduction

Terraform is primarily used for:

```text
Infrastructure Provisioning
```

Examples:

```text
EC2

VPC

Subnets

Load Balancers

Route53

Security Groups
```

---

After infrastructure creation, sometimes we need:

```text
Server Configuration
```

Examples:

```text
Install MongoDB

Install Nginx

Install Docker

Run Ansible Playbooks

Configure Applications
```

Terraform provides:

```text
Provisioners
```

for such tasks.

---

# MongoDB Deployment Example

Suppose we want to deploy MongoDB.

Steps:

```text
1. Create MongoDB EC2 Instance

2. Connect To MongoDB Server

3. Configure MongoDB

4. Run Ansible Playbook
```

---

# Traditional Approach

```text
Terraform
      ↓
Create EC2
      ↓
SSH Login
      ↓
Run Commands
      ↓
Configure Server
```

---

# Bastion Based Deployment

In production:

```text
MongoDB
```

usually resides in:

```text
Private Subnet
```

Direct access is not allowed.

---

Architecture:

```text
Laptop
   ↓
Bastion Host
   ↓
MongoDB
```

---

Terraform Execution:

```text
Terraform Running On Bastion
```

Flow:

```text
Bastion
    ↓
Create MongoDB
    ↓
Configure MongoDB
```

---

# What is a Provisioner?

Provisioners execute commands after resource creation.

Purpose:

```text
Configuration Management

Software Installation

Automation
```

---

# Types of Provisioners

Terraform supports:

```text
local-exec

remote-exec
```

---

# local-exec

Runs commands on:

```text
Local Machine
```

where Terraform is executed.

---

Example:

```hcl
provisioner "local-exec" {

  command = "echo MongoDB Created"

}
```

---

Flow:

```text
Terraform Host
      ↓
Execute Command
```

---

# Production Example

```hcl
provisioner "local-exec" {

  command = "ansible-playbook mongodb.yaml"

}
```

Purpose:

```text
Trigger Ansible Playbook
```

after resource creation.

---

# remote-exec

Runs commands inside:

```text
Remote Server
```

created by Terraform.

---

Example:

```hcl
provisioner "remote-exec" {

  inline = [

    "sudo dnf install nginx -y"

  ]

}
```

---

Flow

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

# MongoDB Example

```text
Terraform
      ↓
Create MongoDB EC2
      ↓
SSH To MongoDB
      ↓
Install MongoDB
```

---

# Production Example

```text
Bastion
      ↓
Terraform Apply
      ↓
MongoDB Created
      ↓
remote-exec
      ↓
MongoDB Configuration
```

---

# Connection Block

remote-exec requires:

```hcl
connection {

}
```

Example:

```hcl
connection {

  type = "ssh"

  user = "ec2-user"

  password = "password"

  host = self.private_ip

}
```

Purpose:

```text
Connect To Remote Server
```

---

# Why Provisioners Are Not Preferred?

Terraform is:

```text
Infrastructure As Code
```

not:

```text
Configuration Management Tool
```

---

Tools better suited for configuration:

```text
Ansible

Chef

Puppet
```

---

Recommended Approach:

```text
Terraform
      ↓
Create Infrastructure
      ↓
Ansible
      ↓
Configure Infrastructure
```

---

# What is null_resource?

Terraform Resource:

```hcl
resource "null_resource" "mongodb" {

}
```

Purpose:

```text
Does Not Create Infrastructure
```

---

# Why Use null_resource?

Used for:

```text
Scripts

Commands

Provisioners

Automation Tasks
```

---

# Example

```hcl
resource "null_resource" "mongodb" {

  provisioner "local-exec" {

    command = "ansible-playbook mongodb.yaml"

  }

}
```

---

# Important Point

null_resource:

```text
Creates No AWS Resource
```

but participates in:

```text
Terraform Lifecycle
```

---

# Lifecycle

null_resource supports:

```text
Create

Update

Destroy
```

like normal Terraform resources.

---

# Production Example

```text
Create EC2
      ↓
Trigger Ansible
      ↓
Configure Application
```

using:

```text
null_resource
```

---

# What is terraform_data?

Terraform introduced:

```text
terraform_data
```

as a modern replacement.

---

Older Method:

```text
null_resource
```

---

Recommended Method:

```text
terraform_data
```

---

# Why terraform_data?

Benefits:

```text
Cleaner Design

Modern Approach

Better Lifecycle Handling
```

---

# Example Use Case

```text
Create MongoDB
      ↓
Run Configuration
      ↓
Run Validation
```

using:

```text
terraform_data
```

---

# MongoDB Bastion Example

Resource Name:

```text
mongodb_bastion
```

Purpose:

```text
Create MongoDB

Configure Through Bastion
```

---

Flow

```text
Bastion
      ↓
Terraform Apply
      ↓
MongoDB Created
      ↓
Ansible Playbook
      ↓
MongoDB Ready
```

---

# Best Practice

Recommended:

```text
Terraform
      ↓
Create Infrastructure
      ↓
Ansible
      ↓
Configuration
```

Avoid:

```text
Large Provisioner Scripts
```

inside Terraform.

---

# Common Interview Questions

## What is a Provisioner?

### Short Answer

A Provisioner executes commands after Terraform creates a resource.

### Detailed Explanation

Provisioners help configure servers, install software, and perform automation tasks after infrastructure creation.

### Production Example

Installing MongoDB packages after creating an EC2 instance.

### Follow-Up Questions

- What are the types of provisioners?
- Why should provisioners be used carefully?

---

## Difference Between local-exec and remote-exec?

### Short Answer

local-exec runs on the Terraform machine, remote-exec runs on the remote server.

### Detailed Explanation

local-exec executes commands where Terraform is running. remote-exec connects to the created resource and executes commands there.

### Production Example

local-exec triggers Ansible, remote-exec installs packages directly on EC2.

### Follow-Up Questions

- Which provisioner requires SSH?
- Which provisioner is preferred with Ansible?

---

## What is null_resource?

### Short Answer

A Terraform resource that creates no infrastructure but can execute provisioners.

### Detailed Explanation

null_resource is used to execute scripts and automation tasks while participating in Terraform lifecycle operations.

### Production Example

Running an Ansible playbook after MongoDB creation.

### Follow-Up Questions

- Does null_resource create AWS resources?
- Why was terraform_data introduced?

---

## What is terraform_data?

### Short Answer

terraform_data is the modern replacement for null_resource.

### Detailed Explanation

It provides a cleaner and more predictable way to execute logic within Terraform workflows.

### Production Example

Triggering validation or configuration tasks after infrastructure deployment.

### Follow-Up Questions

- Difference between terraform_data and null_resource?
- Which one should be used in new projects?

---

## Why Use Ansible Along With Terraform?

### Short Answer

Terraform creates infrastructure and Ansible configures it.

### Detailed Explanation

Terraform is designed for infrastructure provisioning, whereas Ansible specializes in server configuration and application deployment.

### Production Example

Terraform creates MongoDB EC2 and Ansible installs and configures MongoDB.

### Follow-Up Questions

- Why is Terraform not considered a configuration management tool?
- What tasks are better suited for Ansible?

---

# Key Takeaways

```text
Terraform creates infrastructure.

Provisioners execute commands after resource creation.

local-exec runs on the Terraform machine.

remote-exec runs on the target server.

MongoDB can be configured through Bastion using remote-exec.

null_resource creates no infrastructure.

terraform_data is the modern replacement for null_resource.

Terraform and Ansible are commonly used together in production environments.
```