# AWS CodeDeploy Documentation

## Overview

AWS CodeDeploy is an automated deployment service that streamlines the process of deploying applications across various AWS compute platforms. The service supports deployments to Amazon EC2 instances, on-premises servers, AWS Lambda functions, and Amazon ECS services. CodeDeploy provides sophisticated deployment control features, including automated rollback capabilities triggered by deployment failures or CloudWatch alarms. The entire deployment process is defined in an appspec.yml file.

## Platform Support

### EC2 and On-premises Platform

CodeDeploy provides comprehensive support for deploying applications to EC2 instances and on-premises servers. The service supports both in-place and blue/green deployment strategies, with the requirement that target instances run the CodeDeploy Agent.

Deployment speeds can be customized through various options:

- AllAtOnce: Fastest deployment with maximum downtime
- HalfAtATime: Balanced approach with 50% capacity reduction
- OneAtATime: Minimal availability impact with longest deployment time
- Custom: User-defined percentage-based deployment

### Lambda Platform

For Lambda deployments, CodeDeploy automates traffic shifting for Lambda aliases, featuring tight integration with the AWS Serverless Application Model (SAM) framework. Traffic shifting patterns include:

Linear deployments:

- LambdaLinear10PercentEvery3Minutes
- LambdaLinear10PercentEvery10Minutes

Canary deployments:

- LambdaCanary10Percent5Minutes
- LambdaCanary10Percent30Minutes

AllAtOnce deployment for immediate traffic shifting

### ECS Platform

CodeDeploy automates the deployment of new ECS Task Definitions exclusively through blue/green deployments. Traffic shifting patterns include:

Linear deployments:

- ECSLinear10PercentEvery3Minutes
- ECSLinear10PercentEvery10Minutes

Canary deployments:

- ECSCanary10Percent5Minutes
- ECSCanary10Percent30Minutes

AllAtOnce deployment for immediate updates

## CodeDeploy Agent

The CodeDeploy Agent is a crucial component that must be running on target EC2 instances prior to deployment. The agent can be automatically installed and updated using AWS Systems Manager. Instances must have appropriate IAM permissions to access deployment bundles stored in Amazon S3.

## Deployment Configurations

### EC2 Deployment Process
Deployments to EC2 instances are governed by the appspec.yml file and the chosen deployment strategy. The process supports deployment hooks for verification at various phases of the deployment lifecycle.

### Auto Scaling Group Integration

In-place Deployments:

- Updates existing EC2 instances
- Automatically includes newly created instances in the deployment process

Blue/Green Deployments:

- Creates a new Auto Scaling Group with copied settings
- Requires an Elastic Load Balancer
- Allows customization of instance retention period for the old ASG

## Rollback Management

CodeDeploy offers flexible rollback capabilities to maintain application reliability:

Automatic Rollbacks:

- Triggered by deployment failures
- Initiated when CloudWatch Alarm thresholds are exceeded

Manual Rollbacks:

- User-initiated rollback to previous version
- Option to disable rollbacks for specific deployments

When a rollback occurs, CodeDeploy creates a new deployment using the last known good revision rather than restoring a previous version. This approach ensures consistent deployment processes and maintains deployment history.

## Best Practices

- Thoroughly test deployment configurations in non-production environments
- Implement appropriate CloudWatch Alarms for automated rollbacks
- Maintain proper version control of your appspec.yml file
- Regular monitoring and maintenance of the CodeDeploy Agent
- Implement appropriate security controls and IAM permissions
- Use deployment hooks effectively for validation
- Maintain comprehensive documentation of deployment configurations
