# Elastic Container Service (ECS)

## Overview
Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that enables you to run Docker containers on AWS infrastructure. This document provides a comprehensive overview of ECS components, features, and best practices.

## Launch Types

### EC2 Launch Type
ECS with EC2 launch type requires you to provision and maintain your own EC2 instances. Key characteristics include:

* You must provision and maintain the infrastructure (EC2 instances)
* Each EC2 instance must run the ECS Agent to register in the ECS Cluster
* AWS manages container starting and stopping
* Requires more management but offers more control

### Fargate Launch Type
Fargate is a serverless compute engine for containers. Key features include:

* No infrastructure provisioning required
* Fully serverless operation
* Only requires task definitions
* AWS automatically runs ECS Tasks based on CPU/RAM requirements
* Scaling is simplified - just increase the number of tasks

## IAM Roles for ECS

### EC2 Instance Profile (EC2 Launch Type only)
Used by the ECS agent for:

* Making API calls to ECS service
* Sending container logs to CloudWatch Logs
* Pulling Docker images from ECR
* Accessing sensitive data in Secrets Manager or SSM Parameter Store

### ECS Task Role
* Allows each task to have a specific role
* Different roles can be used for different ECS Services
* Defined in the task definition

## Load Balancer Integration

### Supported Load Balancers
* Application Load Balancer (ALB) - recommended for most use cases
* Network Load Balancer (NLB) - recommended for high throughput/performance cases or AWS Private Link
* Classic Load Balancer - supported but not recommended (no advanced features, no Fargate support)

## Data Volumes

### EFS Integration
* Mount EFS file systems onto ECS tasks
* Compatible with both EC2 and Fargate launch types
* Tasks in any AZ share the same data
* Fargate + EFS provides fully serverless solution
* Note: Amazon S3 cannot be mounted as a file system

### Bind Mounts
* Share data between multiple containers in the same Task Definition
* Works for both EC2 and Fargate tasks
* EC2 Tasks: Uses EC2 instance storage (data tied to EC2 instance lifecycle)
* Fargate Tasks: Uses ephemeral storage (20-200 GiB, default 20 GiB)

## Auto Scaling

### Service Auto Scaling
* Automatically adjusts the number of ECS tasks
* Uses AWS Application Auto Scaling
* Scaling metrics:

  - ECS Service Average CPU Utilization
  - ECS Service Average Memory Utilization
  - ALB Request Count PerTarget

### Scaling Methods
* Target Tracking - based on CloudWatch metric target
* Step Scaling - based on CloudWatch Alarm
* Scheduled Scaling - based on date/time

### EC2 Launch Type Auto Scaling
* Auto Scaling Group Scaling based on CPU Utilization
* ECS Cluster Capacity Provider for automatic infrastructure provisioning

## Task Definitions
Task definitions are JSON metadata that tell ECS how to run Docker containers. They include:

* Image Name
* Port Binding for Container and Host
* Memory and CPU requirements
* Environment variables
* Networking information
* IAM Role
* Logging configuration
* Up to 10 containers per Task Definition

## Environment Variables
Supported types:

* Hardcoded values (e.g., URLs)
* SSM Parameter Store (sensitive variables, API keys, configs)
* Secrets Manager (sensitive variables, DB passwords)
* Environment Files (bulk) from Amazon S3

## Task Placement (EC2 Launch Type Only)

### Placement Process
1. Identify instances meeting CPU, memory, and port requirements
2. Identify instances satisfying Task Placement Constraints
3. Identify instances satisfying Task Placement Strategies
4. Select instances

### Placement Strategies
* Binpack - optimizes resource usage by placing tasks on instances with least available CPU/memory
* Random - places tasks randomly across instances
* Spread - distributes tasks evenly based on specified values (e.g., availability zone)

### Placement Constraints
* distinctInstance - ensures tasks are placed on different EC2 instances
* memberOf - places tasks on instances meeting specified criteria using Cluster Query Language

## Rolling Updates
Control task updates from v1 to v2 by specifying:

* Minimum Healthy Percent (0-100%)
* Maximum Percent (100-200%)
* Controls task termination and creation order during updates

## Load Balancing

### EC2 Launch Type
* Supports Dynamic Host Port Mapping
* ALB automatically finds correct ports on EC2 instances
* Security Group configuration required for EC2 instances

### Fargate
* Each task gets unique private IP
* Only container port definition required
* Security Group configuration needed for ECS ENI and ALB
