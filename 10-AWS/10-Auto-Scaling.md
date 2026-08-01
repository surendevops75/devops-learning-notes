# Amazon EC2 Auto Scaling

---

# Introduction

Amazon EC2 Auto Scaling is a fully managed AWS service that automatically adjusts the number of EC2 instances based on application demand.

Instead of manually launching or terminating EC2 instances during traffic fluctuations, Auto Scaling continuously monitors application health and CloudWatch metrics to ensure the desired number of healthy instances are always running.

Auto Scaling helps organizations build applications that are:

- Highly Available
- Fault Tolerant
- Scalable
- Cost Optimized
- Self-Healing

It is one of the most important services used in production AWS environments.

---

# What is Auto Scaling?

Amazon EC2 Auto Scaling automatically launches or terminates EC2 instances based on predefined scaling policies.

Instead of keeping a fixed number of servers running all the time, Auto Scaling adjusts capacity dynamically according to workload demand.

Example

```text
Morning Traffic

↓

2 EC2 Instances

↓

Afternoon Traffic

↓

8 EC2 Instances

↓

Night Traffic

↓

2 EC2 Instances
```

This ensures applications remain responsive while reducing infrastructure costs.

---

# Why Auto Scaling?

Imagine an online shopping application hosted on two EC2 instances.

Normal Days

```text
Users

↓

ALB

↓

EC2-1

EC2-2
```

During a flash sale:

```text
200 Users

↓

20,000 Users
```

Problems:

- High CPU Utilization
- Slow Response Time
- Request Timeouts
- Application Crash

Auto Scaling detects increased demand and automatically launches additional EC2 instances.

```text
Users

↓

ALB

↓

EC2-1

EC2-2

EC2-3

EC2-4

EC2-5
```

Once traffic decreases, unnecessary instances are automatically terminated.

---

# Real-World Problem

An e-commerce company experiences:

- Normal Traffic: 5,000 users/hour
- Festival Traffic: 500,000 users/hour

Requirements:

- No downtime
- Automatic scaling
- Cost optimization
- Zero manual intervention
- High availability

Amazon EC2 Auto Scaling fulfills all these requirements.

---

# Enterprise Architecture

```text
                         Internet

                             │

                        Amazon Route53

                             │

                 Application Load Balancer

                             │

                  Auto Scaling Group (ASG)

        ┌───────────────────┼───────────────────┐

        │                   │                   │

    EC2 Instance       EC2 Instance       EC2 Instance

        │                   │                   │

        └───────────────────┼───────────────────┘

                     CloudWatch Metrics

                             │

                   Scaling Policies

                             │

                Launch / Terminate Instances
```

---

# Internal Working

The Auto Scaling workflow consists of continuous monitoring and automated actions.

```text
CloudWatch Metrics

↓

Scaling Policy Evaluation

↓

Auto Scaling Group

↓

Launch Template

↓

Launch or Terminate EC2

↓

Register with ALB

↓

Serve Traffic
```

This cycle repeats continuously.

---

# Core Components

Amazon EC2 Auto Scaling consists of:

- Launch Template
- Launch Configuration (Legacy)
- Auto Scaling Group
- Scaling Policies
- CloudWatch Alarms
- Health Checks
- Lifecycle Hooks
- Warm Pools
- Instance Refresh

---

# Launch Template

A Launch Template defines how new EC2 instances should be created.

It contains:

- Amazon Machine Image (AMI)
- Instance Type
- Key Pair
- Security Groups
- IAM Role
- User Data
- Storage Configuration
- Network Configuration

Think of a Launch Template as a reusable blueprint for launching EC2 instances.

---

# Why Launch Templates?

Without Launch Templates:

Every EC2 launch requires manual configuration.

With Launch Templates:

```text
Scaling Event

↓

Launch Template

↓

Automatically Create EC2
```

This ensures consistency across all instances.

---

# Launch Template Components

Typical configuration includes:

- AMI ID
- Instance Type
- IAM Instance Profile
- Security Groups
- Root Volume
- User Data Script
- Monitoring
- Tags

---

# Launch Configuration (Legacy)

Launch Configurations were the original mechanism for defining EC2 launch settings.

Limitations:

- Cannot be modified
- Fewer features
- No versioning
- No Mixed Instance support

AWS recommends using Launch Templates for all new deployments.

---

# Auto Scaling Group (ASG)

An Auto Scaling Group manages a collection of EC2 instances.

Responsibilities:

- Launch instances
- Terminate instances
- Replace unhealthy instances
- Maintain desired capacity
- Distribute instances across Availability Zones

Architecture

```text
Auto Scaling Group

↓

Launch Template

↓

EC2-1

EC2-2

EC2-3
```

---

# Desired Capacity

Desired Capacity defines how many EC2 instances should currently be running.

Example

```
Desired = 3
```

If one instance fails:

```text
Running

↓

2 Instances

↓

Auto Scaling

↓

Launch New Instance

↓

Back to 3
```

This self-healing capability is one of the biggest advantages of Auto Scaling.

---

# Minimum Capacity

Defines the minimum number of instances that must always remain running.

Example

```
Minimum = 2
```

Even if traffic drops to zero:

```
2 EC2 Instances

Remain Running
```

---

# Maximum Capacity

Defines the maximum number of instances Auto Scaling is allowed to launch.

Example

```
Maximum = 10
```

Even during heavy traffic:

```
Never Exceeds

10 EC2 Instances
```

---

# Capacity Relationship

Example

```
Minimum = 2

Desired = 4

Maximum = 10
```

Rules:

- Running instances should be 4
- Never go below 2
- Never exceed 10

---

# Scaling Policies

Scaling Policies determine when Auto Scaling should launch or terminate instances.

AWS supports:

- Target Tracking Scaling
- Step Scaling
- Simple Scaling
- Scheduled Scaling
- Predictive Scaling

---

# Target Tracking Scaling

The most commonly used scaling policy.

You specify a target metric.

Example

```
Target CPU

50%
```

If CPU rises above 50%:

```
Launch EC2
```

If CPU falls below 50%:

```
Terminate EC2
```

AWS automatically calculates how many instances should be added or removed.

Recommended for most production workloads.

---

# Step Scaling

Step Scaling performs different scaling actions depending on metric thresholds.

Example

| CPU Utilization | Action |
|-----------------|--------|
| 60–70% | Add 1 Instance |
| 70–80% | Add 2 Instances |
| Above 80% | Add 4 Instances |

Useful when workloads increase rapidly.

---

# Simple Scaling

Simple Scaling performs one scaling action after a CloudWatch alarm is triggered.

Example

```
CPU > 70%

↓

Launch 1 EC2

↓

Cooldown Period
```

This policy is easier to configure but slower than Target Tracking.

---

# Scheduled Scaling

Scheduled Scaling adjusts capacity at specific times.

Example

```text
Monday

08:00 AM

↓

Increase to 10 Instances

-------------------------

11:00 PM

↓

Reduce to 2 Instances
```

Ideal for predictable workloads.

Examples:

- Office Applications
- Batch Processing
- Business Hours Traffic

---

# Predictive Scaling

Predictive Scaling uses machine learning to forecast future traffic.

Instead of reacting after CPU increases:

```
Traffic Predicted

↓

Launch EC2

↓

Traffic Arrives
```

Benefits

- Faster response
- Better performance
- Reduced latency

Suitable for applications with predictable usage patterns.

---

# Dynamic Scaling

Dynamic Scaling automatically adjusts capacity based on real-time CloudWatch metrics.

Metrics commonly used:

- CPU Utilization
- Memory Utilization (Custom Metric)
- Request Count
- Network In
- Network Out
- ALB Request Count Per Target

Example

```text
CloudWatch

↓

CPU 75%

↓

Scaling Policy

↓

Launch EC2

↓

Register with ALB
```

---

# Cooldown Period

After a scaling activity completes, Auto Scaling waits before performing another scaling action.

Purpose:

- Prevent rapid scaling
- Avoid unnecessary instance launches
- Allow applications time to stabilize

Example

```
Scale Out

↓

Wait

300 Seconds

↓

Evaluate Again
```

---

# Health Checks

Auto Scaling continuously monitors EC2 instance health.

Supports two health check types:

- EC2 Health Check
- ELB Health Check

If an instance becomes unhealthy:

```text
EC2 Failed

↓

Terminate

↓

Launch Replacement

↓

Register with ALB
```

Applications remain available without manual intervention.

---

# EC2 Health Check

Checks whether the EC2 instance itself is healthy.

Examples:

- Instance Status
- System Status
- Hardware Failure
- Operating System Crash

---

# Elastic Load Balancer Health Check

ELB verifies that the application is actually responding.

Example

```
GET /health

↓

HTTP 200

↓

Healthy
```

If the application fails but the EC2 instance is still running, ELB marks it unhealthy and Auto Scaling replaces it.

---

# Lifecycle Hooks

Lifecycle Hooks allow you to pause an instance during launch or termination.

Launch Example

```text
Launch EC2

↓

Wait

↓

Install Software

↓

Configuration

↓

Complete Lifecycle

↓

Register with ALB
```

Termination Example

```text
Terminate Instance

↓

Pause

↓

Upload Logs

↓

Backup Data

↓

Complete

↓

Terminate
```

Lifecycle Hooks are commonly used for:

- Configuration Management
- Log Collection
- Backup
- Notifications
- Monitoring

---

---

# Instance Refresh

Instance Refresh allows you to gradually replace EC2 instances in an Auto Scaling Group without creating a new ASG.

Common use cases:

- Deploying a new AMI
- Updating User Data
- Installing a new application version
- Updating security patches
- Changing the instance type

Workflow

```text
Update Launch Template

↓

Start Instance Refresh

↓

Launch New Instance

↓

Health Check

↓

Register with ALB

↓

Terminate Old Instance

↓

Repeat Until Complete
```

Benefits

- Zero downtime deployments
- Controlled rollout
- Automatic rollback (when configured)
- No manual instance replacement

---

# Warm Pools

Warm Pools keep pre-initialized EC2 instances ready for faster scaling.

Without Warm Pool

```text
Traffic Spike

↓

Launch EC2

↓

Boot OS

↓

Install Packages

↓

Application Starts

↓

Receive Traffic
```

This may take several minutes.

With Warm Pool

```text
Traffic Spike

↓

Move Warm Instance

↓

Health Check

↓

Receive Traffic
```

Benefits

- Faster scale-out
- Reduced startup time
- Better user experience

---

# Mixed Instance Policy

A Mixed Instance Policy allows an Auto Scaling Group to launch multiple EC2 instance types.

Example

```text
Auto Scaling Group

├── t3.large

├── t3a.large

├── m5.large

├── m6i.large

└── c6i.large
```

Benefits

- Better availability
- Improved capacity acquisition
- Lower cost
- Flexible instance selection

---

# Spot and On-Demand Instances

Auto Scaling can combine Spot and On-Demand instances.

Example

```text
Desired Capacity = 10

↓

7 On-Demand

3 Spot
```

Advantages

- Significant cost savings
- High availability
- Flexible capacity

Typical Production Strategy

- Baseline capacity → On-Demand
- Additional capacity → Spot

---

# Capacity Rebalancing

Spot Instances can be interrupted by AWS.

Capacity Rebalancing proactively launches replacement Spot instances before interruption occurs.

Workflow

```text
Spot Interruption Notice

↓

Launch Replacement Instance

↓

Register with ALB

↓

Drain Existing Instance

↓

Terminate
```

This minimizes application disruption.

---

# Standby Mode

Standby Mode temporarily removes an instance from serving traffic without terminating it.

Use Cases

- Troubleshooting
- Maintenance
- Debugging
- Performance Testing

Workflow

```text
EC2

↓

Standby

↓

Removed from ALB

↓

Maintenance

↓

Return to Service
```

---

# Scale-In Protection

Scale-In Protection prevents selected instances from being terminated during scale-in events.

Useful for:

- Critical application nodes
- Long-running batch jobs
- Stateful workloads

---

# Termination Policies

When scaling in, Auto Scaling decides which instance to terminate.

Common policies:

- Oldest Instance
- Oldest Launch Template
- Closest to Billing Hour (legacy)
- Default Policy
- Oldest Launch Configuration

AWS selects the most appropriate instance based on the configured policy.

---

# CloudWatch Integration

Amazon EC2 Auto Scaling relies heavily on CloudWatch metrics.

Common Metrics

- CPU Utilization
- Network In
- Network Out
- ALB Request Count Per Target
- Custom Memory Metrics
- Disk Utilization (Custom)

Workflow

```text
CloudWatch

↓

Metric Threshold

↓

Alarm

↓

Scaling Policy

↓

Auto Scaling Group
```

---

# Integration with Elastic Load Balancer

Elastic Load Balancer and Auto Scaling work together to provide high availability.

Workflow

```text
Users

↓

Application Load Balancer

↓

Target Group

↓

Auto Scaling Group

↓

EC2 Instances
```

When a new instance launches:

```text
Launch Instance

↓

Health Check

↓

Register with Target Group

↓

Receive Traffic
```

When an instance terminates:

```text
Deregister

↓

Connection Draining

↓

Terminate
```

---

# Integration with Amazon EKS

Amazon EKS worker nodes are often managed using Auto Scaling Groups.

Architecture

```text
Amazon EKS

↓

Managed Node Group

↓

Auto Scaling Group

↓

EC2 Worker Nodes

↓

Pods
```

Benefits

- Automatic node replacement
- High availability
- Cluster scalability

---

# Enterprise Production Architecture

```text
                           Internet

                               │

                           Route53

                               │

                  Application Load Balancer

                               │

                  Auto Scaling Group (ASG)

       ┌───────────────────────┼────────────────────────┐

       │                       │                        │

   EC2 (AZ-A)             EC2 (AZ-B)              EC2 (AZ-C)

       │                       │                        │

       └───────────────────────┼────────────────────────┘

                       Amazon RDS Multi-AZ

                               │

                         CloudWatch

                               │

                     Scaling Policies

                               │

                    Launch Template
```

---

# AWS Console Walkthrough

1. Open EC2 Console
2. Select **Auto Scaling Groups**
3. Click **Create Auto Scaling Group**
4. Select Launch Template
5. Choose VPC and Subnets
6. Attach Load Balancer
7. Configure Health Checks
8. Set Minimum, Desired, and Maximum Capacity
9. Configure Scaling Policies
10. Review and Create

---

# AWS CLI

Create Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
--auto-scaling-group-name production-asg \
--launch-template LaunchTemplateName=web-template \
--min-size 2 \
--max-size 10 \
--desired-capacity 3 \
--vpc-zone-identifier subnet-123,subnet-456
```

Describe Auto Scaling Groups

```bash
aws autoscaling describe-auto-scaling-groups
```

Update Desired Capacity

```bash
aws autoscaling set-desired-capacity \
--auto-scaling-group-name production-asg \
--desired-capacity 5
```

Start Instance Refresh

```bash
aws autoscaling start-instance-refresh \
--auto-scaling-group-name production-asg
```

Delete Auto Scaling Group

```bash
aws autoscaling delete-auto-scaling-group \
--auto-scaling-group-name production-asg \
--force-delete
```

---

# Terraform

Launch Template

```hcl
resource "aws_launch_template" "web" {

  name_prefix = "web-template"

  image_id = "ami-xxxxxxxx"

  instance_type = "t3.medium"

}
```

Auto Scaling Group

```hcl
resource "aws_autoscaling_group" "web" {

  desired_capacity = 3

  min_size = 2

  max_size = 10

  vpc_zone_identifier = [

    aws_subnet.private_a.id,

    aws_subnet.private_b.id

  ]

  launch_template {

    id = aws_launch_template.web.id

    version = "$Latest"

  }

}
```

Target Tracking Policy

```hcl
resource "aws_autoscaling_policy" "cpu" {

  name = "cpu-policy"

  autoscaling_group_name = aws_autoscaling_group.web.name

  policy_type = "TargetTrackingScaling"

  target_tracking_configuration {

    predefined_metric_specification {

      predefined_metric_type = "ASGAverageCPUUtilization"

    }

    target_value = 50

  }

}
```

---

# CloudFormation

```yaml
Resources:

  AutoScalingGroup:

    Type: AWS::AutoScaling::AutoScalingGroup

    Properties:

      MinSize: 2

      MaxSize: 10

      DesiredCapacity: 3
```

---

# Python (Boto3)

```python
import boto3

client = boto3.client("autoscaling")

response = client.describe_auto_scaling_groups()

print(response)
```

---

# Best Practices

- Use Launch Templates instead of Launch Configurations
- Distribute instances across multiple Availability Zones
- Integrate with Application Load Balancer
- Configure ELB Health Checks
- Use Target Tracking for most workloads
- Enable Instance Refresh for deployments
- Use Warm Pools for slow-starting applications
- Combine Spot and On-Demand instances
- Enable detailed CloudWatch monitoring
- Tag Auto Scaling resources consistently

---

# Common Mistakes

- Setting Minimum Capacity to 0 for production
- Using only one Availability Zone
- Not attaching an ALB
- Missing Health Checks
- Hardcoding Desired Capacity
- Ignoring Cooldown periods
- Not updating Launch Templates
- Using Launch Configurations for new deployments

---

# Troubleshooting

## Auto Scaling Does Not Launch Instances

Check:

- Launch Template
- AMI
- IAM Role
- EC2 Limits
- Subnet Capacity
- Scaling Policy

---

## Instances Launch but Receive No Traffic

Verify:

- Target Group Registration
- ALB Health Checks
- Security Groups
- Listener Rules

---

## Auto Scaling Never Scales Out

Check:

- CloudWatch Alarm
- Target Tracking Configuration
- CPU Metrics
- Desired Capacity
- Maximum Capacity

---

## Continuous Scale In and Scale Out

Possible causes:

- Incorrect Target Value
- Short Cooldown
- Fluctuating Metrics
- Poor Scaling Policy Design

---

## Instance Refresh Fails

Verify:

- Launch Template Version
- Health Checks
- Minimum Healthy Percentage
- Target Group Status

---

# Interview Questions

### Basic

1. What is Amazon EC2 Auto Scaling?
2. Why do we use Auto Scaling?
3. What is an Auto Scaling Group?
4. What is a Launch Template?
5. Difference between Launch Template and Launch Configuration?
6. What is Desired Capacity?
7. What is Minimum Capacity?
8. What is Maximum Capacity?
9. What is a Scaling Policy?
10. What is Target Tracking Scaling?

### Intermediate

11. Explain Step Scaling.
12. Explain Simple Scaling.
13. Explain Scheduled Scaling.
14. Explain Predictive Scaling.
15. What are Lifecycle Hooks?
16. What are Warm Pools?
17. What is Instance Refresh?
18. Explain ELB Health Checks.
19. What is Scale-In Protection?
20. What is Capacity Rebalancing?

### Advanced

21. How does Auto Scaling integrate with ALB?
22. How does Auto Scaling integrate with CloudWatch?
23. How do Mixed Instance Policies work?
24. How do Spot Instances reduce cost?
25. Explain Auto Scaling in Amazon EKS.
26. How do you troubleshoot scaling failures?
27. What causes scaling oscillation?
28. How do you perform zero-downtime deployments?
29. Design a highly available Auto Scaling architecture.
30. Explain the complete Auto Scaling lifecycle.

---

# Scenario-Based Questions

### Scenario 1

CPU utilization remains above 90%, but Auto Scaling does not launch new instances.

How would you troubleshoot?

---

### Scenario 2

New EC2 instances launch successfully but never receive production traffic.

What components would you verify?

---

### Scenario 3

A deployment requires replacing every EC2 instance with a new AMI without downtime.

Which Auto Scaling feature would you use?

---

### Scenario 4

Your application startup takes 6 minutes, causing slow scale-out during traffic spikes.

How would you improve scaling performance?

---

### Scenario 5

Spot Instances are frequently interrupted during peak hours.

How can Capacity Rebalancing help?

---

### Scenario 6

An Auto Scaling Group is continuously launching and terminating instances every few minutes.

What configuration issues might cause this behavior?

---

### Scenario 7

Your organization wants to reduce EC2 costs while maintaining production availability.

How would you design the Auto Scaling Group?

---

### Scenario 8

One Availability Zone becomes unavailable.

How does Auto Scaling maintain application availability?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Launch Template | Blueprint for EC2 instances |
| Auto Scaling Group | Manages EC2 instances |
| Target Tracking | Maintains target metric |
| Step Scaling | Scales based on thresholds |
| Scheduled Scaling | Time-based scaling |
| Predictive Scaling | Forecast-based scaling |
| Lifecycle Hook | Pause launch/termination |
| Warm Pool | Pre-initialized instances |
| Instance Refresh | Rolling instance replacement |
| Capacity Rebalancing | Replaces interrupted Spot instances |

---

# Summary

Amazon EC2 Auto Scaling automatically adjusts compute capacity to match application demand while maintaining availability and optimizing costs. By combining Launch Templates, Auto Scaling Groups, CloudWatch metrics, scaling policies, and Elastic Load Balancers, organizations can build resilient, self-healing infrastructure that responds dynamically to changing workloads.

For production environments, use Launch Templates, Target Tracking Scaling, Multi-AZ deployments, ELB health checks, Instance Refresh, Warm Pools for slow-start applications, and a balanced mix of On-Demand and Spot Instances to achieve both reliability and cost efficiency.