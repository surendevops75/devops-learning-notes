# Terraform and Ansible Deployment Automation

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

After infrastructure creation, servers need configuration.

Examples:

```text
Install MongoDB

Install Nginx

Install Docker

Deploy Application

Configure Services
```

Terraform and Ansible are commonly used together to automate this process.

---

# Typical Deployment Flow

```text
Terraform
      ↓
Create Infrastructure
      ↓
Ansible
      ↓
Configure Infrastructure
      ↓
Application Ready
```

---

# Installing Ansible Server

Before deployment automation:

```text
Install Ansible
```

Example:

```bash
sudo dnf install ansible -y
```

or

```bash
pip install ansible
```

---

# Repository Structure

Create a dedicated directory:

```text
/ansible
```

Purpose:

```text
Store Playbooks

Store Inventories

Store Roles

Store Deployment Scripts
```

---

# Log Directory

Create a log directory:

```text
/ansible/logs
```

Purpose:

```text
Store Deployment Logs

Troubleshooting

Audit Tracking
```

---

# Deployment Workflow

Before running deployment:

Check whether repository exists.

---

# Repository Exists?

```text
YES
```

Action:

```bash
git pull
```

Purpose:

```text
Fetch Latest Changes
```

---

# Repository Does Not Exist?

```text
NO
```

Action:

```bash
git clone <repository-url>
```

Purpose:

```text
Download Repository
```

---

# Automation Logic

```text
Check Repository
       ↓
Exists?
       ↓
Yes → git pull

No  → git clone
```

---

# Production Example

```bash
if [ -d "/ansible/roboshop-ansible" ]
then
    cd /ansible/roboshop-ansible
    git pull
else
    git clone https://github.com/company/roboshop-ansible.git
fi
```

---

# Terraform to Shell Integration

Terraform can execute shell scripts.

Flow:

```text
Terraform
      ↓
Shell Script
```

---

# Example

Terraform passes environment:

```text
dev
```

to shell script.

---

Terraform:

```text
terraform apply
```

↓

```text
deploy.sh dev catalogue
```

---

# Shell Script Variables

Inside shell:

```bash
$1
$2
$3
```

represent arguments.

Example:

```bash
deploy.sh dev catalogue
```

Values:

```text
$1 = dev

$2 = catalogue
```

---

# Production Example

Terraform passes:

```text
Environment

Component
```

to deployment script.

---

# Shell to Ansible Integration

Shell script can invoke Ansible.

Flow:

```text
Terraform
      ↓
Shell
      ↓
Ansible
```

---

# Example

Shell receives:

```text
dev

catalogue
```

and passes them to Ansible.

---

Command:

```bash
ansible-playbook \
-e env=dev \
-e component=catalogue \
catalogue.yaml
```

---

# Complete Deployment Flow

```text
Terraform
      ↓
Shell Script
      ↓
Ansible Playbook
      ↓
Configure Server
```

---

# Creating Catalogue Infrastructure

Requirement:

```text
Create Catalogue Instance
```

Steps:

```text
Create EC2

Configure EC2

Deploy Application
```

---

# Terraform Creates EC2

```text
Terraform
      ↓
EC2 Created
```

---

# Ansible Configures EC2

```text
Install Packages

Configure Service

Deploy Application

Start Service
```

---

# Inventory File

Ansible needs target servers.

Example:

```ini
[catalogue]

10.0.11.10
10.0.11.11
10.0.11.12
```

---

# Target Group Example

Target Group:

```text
Catalogue
```

contains:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

# Inventory Mapping

```ini
[catalogue]

catalogue-ip1
catalogue-ip2
catalogue-ip3
```

---

# Production Example

10 Catalogue Servers Running:

```text
Catalogue-1

Catalogue-2

Catalogue-3

...

Catalogue-10
```

Ansible can connect to all servers simultaneously and deploy a new application version.

---

# Benefits of Ansible

```text
Automation

Consistency

Scalability

Reduced Manual Effort

Faster Deployments
```

---

# Terraform + Ansible Architecture

```text
Terraform
      ↓
Create Infrastructure
      ↓
Shell Script
      ↓
Ansible
      ↓
Server Configuration
      ↓
Application Deployment
```

---

# Production Scenario

New Catalogue Version Released.

Instead of:

```text
SSH To 10 Servers

Deploy Manually
```

Use:

```text
Ansible Playbook
```

to deploy everywhere automatically.

---

# Common Interview Questions

## Why Use Terraform and Ansible Together?

### Short Answer

Terraform creates infrastructure and Ansible configures infrastructure.

### Detailed Explanation

Terraform is designed for infrastructure provisioning while Ansible specializes in configuration management and application deployment.

### Production Example

Terraform creates Catalogue EC2 instances and Ansible installs the application and dependencies.

### Follow-Up Questions

- Why is Terraform not considered a configuration management tool?
- What tasks are better suited for Ansible?

---

## What is an Ansible Inventory?

### Short Answer

An inventory contains the target servers managed by Ansible.

### Detailed Explanation

Inventory files define hosts and groups of hosts that Ansible should manage.

### Production Example

```ini
[catalogue]

10.0.11.10
10.0.11.11
10.0.11.12
```

### Follow-Up Questions

- Can inventory contain groups?
- What is a dynamic inventory?

---

## How Does Terraform Trigger Ansible?

### Short Answer

Terraform can execute shell scripts or provisioners that invoke Ansible playbooks.

### Detailed Explanation

Terraform passes parameters to shell scripts, and shell scripts execute Ansible playbooks with the required variables.

### Production Example

```text
Terraform
      ↓
deploy.sh
      ↓
ansible-playbook
```

### Follow-Up Questions

- Can Terraform directly run Ansible?
- What are the disadvantages of using provisioners?

---

## Why Check git pull Before Deployment?

### Short Answer

To ensure the latest deployment code is available.

### Detailed Explanation

If the repository already exists, pulling the latest changes avoids unnecessary cloning and ensures deployments use the newest version.

### Production Example

Deployment server updates Roboshop playbooks using:

```bash
git pull
```

before executing Ansible.

### Follow-Up Questions

- Why not clone every time?
- What happens if deployment scripts are outdated?

---

# Key Takeaways

```text
Terraform creates infrastructure.

Ansible configures infrastructure.

Terraform and Ansible complement each other.

Deployment repositories should be cloned or updated automatically.

Shell scripts act as a bridge between Terraform and Ansible.

Ansible inventories define target servers.

Ansible can deploy applications to multiple servers simultaneously.

Automation reduces manual effort and deployment errors.

Terraform → Shell → Ansible is a common production workflow.
```