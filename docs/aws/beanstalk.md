# AWS Elastic Beanstalk

## Introduction

AWS Elastic Beanstalk is a Platform as a Service (PaaS) offering that facilitates quick deployment and management of web applications. It automatically handles infrastructure details like capacity provisioning, load balancing, auto-scaling, and application health monitoring while giving you full control over the AWS resources powering your application.

## Core Concepts

### Environment Tiers
Elastic Beanstalk provides two environment tiers:

1. **Web Server Environment Tier**
- Handles HTTP(S) requests
- Suitable for web applications
- Uses services like ELB, Auto Scaling, EC2

2. **Worker Environment Tier**
- Processes background tasks
- Handles Amazon SQS queues
- Long-running processes

### Supported Platforms

Elastic Beanstalk supports multiple platforms:
- Java (with Tomcat or without)
- .NET (on Windows Server)
- PHP
- Node.js
- Python
- Ruby
- Go
- Docker
- Custom Platforms

## Components

### Application
An application is a collection of Elastic Beanstalk components:
- Environments
- Versions
- Environment configurations
- Platform versions

### Environment
An environment is a collection of AWS resources running an application version:
- EC2 instances
- Auto Scaling group
- Elastic Load Balancer
- Security groups
- CloudWatch alarms

## Deployment Strategies

### 1. All at Once Deployment
```yaml
DeploymentPolicy: AllAtOnce
```

**Characteristics:**
- Deploys to all instances simultaneously
- Fastest deployment method
- Results in total application downtime
- No additional cost
- Fastest rollback

**Best for:**
- Development environments
- Quick iterations
- Non-critical applications

### 2. Rolling Deployment
```yaml
DeploymentPolicy: Rolling
RollingUpdateType: Health
MaxBatchSize: 2
MinInstancesInService: 1
```

**Characteristics:**
- Updates a few instances at a time
- Maintains service availability
- No additional cost
- Longer deployment time
- Runs both versions simultaneously during deployment

**Best for:**
- Production environments
- Applications requiring high availability
- Services with stateless architecture

### 3. Rolling with Additional Batch
```yaml
DeploymentPolicy: RollingWithAdditionalBatch
MaxBatchSize: 2
MinInstancesInService: 2
```

**Characteristics:**
- Launches new instances before removing old ones
- Maintains full capacity during deployment
- Incurs additional costs during deployment
- Better performance during deployment
- Longer deployment time

**Best for:**
- Production environments requiring consistent capacity
- Applications with high traffic requirements
- Critical business applications

### 4. Immutable Deployment
```yaml
DeploymentPolicy: Immutable
```

**Characteristics:**
- Creates new Auto Scaling group with new instances
- Zero downtime
- Safest deployment method
- Highest cost
- Longest deployment time
- Best rollback process

**Best for:**
- Mission-critical production applications
- Applications requiring zero downtime
- Systems requiring guaranteed rollback capability

### 5. Blue/Green Deployment
**Characteristics:**
- Creates entirely new environment
- Zero downtime
- Easy rollback
- Requires DNS update
- Highest cost
- Testing in production-like environment

**Best for:**
- Production environments requiring zero downtime
- Applications with critical uptime requirements
- Systems requiring thorough testing before switch

## Configuration

### Environment Configuration

```yaml
option_settings:
  aws:autoscaling:launchconfiguration:
    InstanceType: t2.micro
    SecurityGroups: my-security-group
  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 4
  aws:elasticbeanstalk:environment:
    EnvironmentType: LoadBalanced
```

### Environment Variables

```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    DB_HOST: mydb.123456789012.us-west-2.rds.amazonaws.com
    DB_PORT: 3306
    API_KEY: ${PARAM_STORE_API_KEY}
```

## Monitoring and Health Checking

### Basic Health Reporting
- System status checks
- Instance health
- Environment health status

### Enhanced Health Reporting
```yaml
option_settings:
  aws:elasticbeanstalk:healthreporting:system:
    SystemType: enhanced
```

Features:
- Operating system metrics
- Application metrics
- Server logs analysis
- Request metrics

## Best Practices

### Application Development
- Use `.ebextensions` for configuration
- Implement proper logging
- Use environment variables
- Implement health checks
- Handle application state properly

### Deployment
- Use immutable deployments for production
- Implement proper rollback procedures
- Use environment cloning for testing
- Maintain version labels
- Use deployment manifest

### Security
- Use security groups effectively
- Implement proper IAM roles
- Use SSL/TLS for communication
- Secure environment variables
- Regular security updates

### Cost Optimization
- Right-size instances
- Use Auto Scaling effectively
- Clean up unused environments
- Monitor resource usage
- Use spot instances where appropriate

## Troubleshooting

### Common Issues

1. **Deployment Failures**
   - Check application logs
   - Verify environment configuration
   - Review instance logs
   - Check permissions

2. **Health Check Failures**
   - Verify application health check path
   - Check instance resources
   - Review application logs
   - Monitor response times

3. **Configuration Issues**
   - Validate `.ebextensions`
   - Check environment variables
   - Review platform version
   - Verify resource limits

## Integration with Other AWS Services

- Amazon RDS
- Amazon S3
- Amazon CloudWatch
- AWS X-Ray
- Amazon VPC
- AWS CodePipeline
- AWS CodeBuild
- AWS Certificate Manager

## Sample Configurations

### Basic Web Application
```yaml
option_settings:
  aws:autoscaling:launchconfiguration:
    InstanceType: t2.micro
  aws:autoscaling:asg:
    MinSize: 1
    MaxSize: 3
  aws:elasticbeanstalk:environment:
    EnvironmentType: LoadBalanced
  aws:elasticbeanstalk:application:environment:
    ENVIRONMENT: production
```

### Worker Environment
```yaml
option_settings:
  aws:autoscaling:launchconfiguration:
    InstanceType: t2.micro
  aws:elasticbeanstalk:sqsd:
    WorkerQueueURL: https://sqs.region.amazonaws.com/account-id/queue-name
    HttpPath: /process-task
```

## CLI Commands

```bash
# Create new application
eb init

# Create new environment
eb create production-env

# Deploy new version
eb deploy

# View environment status
eb status

# View logs
eb logs

# Terminate environment
eb terminate
```

## Resources

- [Official AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elastic-beanstalk/)
- [AWS Elastic Beanstalk Developer Guide](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/)
- [Elastic Beanstalk CLI Documentation](https://docs.aws.amazon.com/elasticbeanstalk/latest/cli/)
- [Sample Applications](https://docs.aws.amazon.com/elastic-beanstalk/latest/dg/tutorials.html)
