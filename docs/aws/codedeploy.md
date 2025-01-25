# CodeDeploy

## Overview

AWS CodeDeploy is a fully managed deployment service that automates software deployments to various compute services including Amazon EC2 instances, AWS Lambda functions, and on-premises servers.

## Core Concepts

### 1. Deployment Components

#### Application
- Container for deployment rules, configurations, and files
- Groups related deployment resources
- Can contain multiple deployment groups

#### Deployment Group
- Set of tagged instances
- Deployment rules and success conditions
- Deployment configuration
- Rollback configuration
- Triggers and alarms

#### Deployment Configuration
- Rules for deployment success/failure
- Traffic shifting rules
- Instance health evaluation

#### Revision
- Source content to deploy
- AppSpec file
- Application files
- Scripts
- Configuration files

### 2. AppSpec File

The AppSpec file (application specification file) is the core of CodeDeploy deployment.

#### Structure
```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/
hooks:
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_application.sh
      timeout: 300
      runas: root
  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
      runas: root
```

#### Lifecycle Events
1. **Application Stop**
   - Stop running version
   - Prepare for deployment

2. **Download Bundle**
   - Download revision files
   - Validate checksums

3. **Before Install**
   - Pre-installation tasks
   - Backup existing files
   - Decrypt files

4. **Install**
   - Copy revision files
   - Set permissions
   - Create directories

5. **After Install**
   - Configuration tasks
   - Set environment variables
   - File modifications

6. **Application Start**
   - Start application
   - Enable services
   - Start servers

7. **Validate Service**
   - Health checks
   - Test functionality
   - Verify deployment

## Deployment Types

### 1. In-Place Deployment

```yaml
deployment_style:
  deployment_type: IN_PLACE
  deployment_option: WITH_TRAFFIC_CONTROL
```

#### Characteristics
- Updates existing instances
- Sequential deployment
- Possible downtime
- Rollback replaces files

#### Use Cases
- Small applications
- Development environments
- Quick updates

### 2. Blue/Green Deployment

```yaml
deployment_style:
  deployment_type: BLUE_GREEN
  deployment_option: WITH_TRAFFIC_CONTROL
```

#### Characteristics
- New instance group
- Zero downtime
- Easy rollback
- Higher resource usage

#### Implementation
```json
{
    "deploymentStyle": {
        "deploymentType": "BLUE_GREEN",
        "deploymentOption": "WITH_TRAFFIC_CONTROL"
    },
    "blueGreenDeploymentConfiguration": {
        "deploymentReadyOption": {
            "actionOnTimeout": "CONTINUE_DEPLOYMENT",
            "waitTimeInMinutes": 60
        },
        "terminateBlueInstancesOnDeploymentSuccess": {
            "action": "TERMINATE",
            "terminationWaitTimeInMinutes": 60
        }
    }
}
```

## Deployment Configurations

### 1. EC2/On-Premises

#### OneAtATime
```json
{
    "deploymentConfigName": "CodeDeployDefault.OneAtATime",
    "minimumHealthyHosts": {
        "type": "HOST_COUNT",
        "value": 1
    }
}
```

#### HalfAtATime
```json
{
    "deploymentConfigName": "CodeDeployDefault.HalfAtATime",
    "minimumHealthyHosts": {
        "type": "FLEET_PERCENT",
        "value": 50
    }
}
```

#### AllAtOnce
```json
{
    "deploymentConfigName": "CodeDeployDefault.AllAtOnce",
    "minimumHealthyHosts": {
        "type": "HOST_COUNT",
        "value": 0
    }
}
```

### 2. Lambda/ECS

#### Linear
```json
{
    "deploymentConfigName": "CodeDeployDefault.LambdaLinear10PercentEvery1Minute",
    "trafficRoutingConfig": {
        "type": "TimeBasedLinear",
        "timeBasedLinear": {
            "linearPercentage": 10,
            "intervalInMinutes": 1
        }
    }
}
```

#### Canary
```json
{
    "deploymentConfigName": "CodeDeployDefault.LambdaCanary10Percent5Minutes",
    "trafficRoutingConfig": {
        "type": "TimeBasedCanary",
        "timeBasedCanary": {
            "canaryPercentage": 10,
            "canaryInterval": 5
        }
    }
}
```

## Integration and Automation

### 1. AWS CLI Commands

#### Create Application
```bash
aws deploy create-application --application-name MyApp

aws deploy create-deployment-group \
    --application-name MyApp \
    --deployment-group-name MyDeploymentGroup \
    --deployment-config-name CodeDeployDefault.OneAtATime \
    --ec2-tag-filters Key=Environment,Value=Production,Type=KEY_AND_VALUE \
    --service-role-arn arn:aws:iam::123456789012:role/CodeDeployServiceRole
```

#### Create Deployment
```bash
aws deploy create-deployment \
    --application-name MyApp \
    --deployment-group-name MyDeploymentGroup \
    --revision {
        "revisionType": "S3",
        "s3Location": {
            "bucket": "my-bucket",
            "key": "revision.zip",
            "bundleType": "zip"
        }
    }
```

### 2. CloudWatch Events Integration

```json
{
    "source": ["aws.codedeploy"],
    "detail-type": ["CodeDeploy Deployment State-change Notification"],
    "detail": {
        "state": ["SUCCESS", "FAILURE"],
        "application": ["MyApp"],
        "deploymentGroup": ["MyDeploymentGroup"]
    }
}
```

## Monitoring and Troubleshooting

### 1. CloudWatch Metrics

#### Available Metrics
- HostCountConnected
- InstanceCount
- DeploymentSuccess
- DeploymentFailure
- DeploymentTime

#### Alarms Configuration
```json
{
    "AlarmName": "DeploymentFailure",
    "ComparisonOperator": "GreaterThanThreshold",
    "EvaluationPeriods": 1,
    "MetricName": "DeploymentFailure",
    "Namespace": "AWS/CodeDeploy",
    "Period": 300,
    "Statistic": "Sum",
    "Threshold": 0,
    "Dimensions": [
        {
            "Name": "Application",
            "Value": "MyApp"
        },
        {
            "Name": "DeploymentGroup",
            "Value": "MyDeploymentGroup"
        }
    ]
}
```

### 2. Logs and Diagnostics

#### Agent Logs
- Location: `/var/log/aws/codedeploy-agent/`
- Debug logs
- Error messages
- Deployment steps

#### Deployment Logs
- Location: `/opt/codedeploy-agent/deployment-root/deployment-logs/`
- Script outputs
- Command results
- Error details

## Best Practices

### 1. Security
1. Use IAM roles and policies
2. Encrypt sensitive data
3. Use secure connections
4. Regular security updates
5. Audit deployment access

### 2. Performance
1. Optimize bundle size
2. Script timeout settings
3. Health check configuration
4. Resource allocation
5. Cache management

### 3. Reliability
1. Implement rollback triggers
2. Test deployments thoroughly
3. Monitor deployments
4. Use deployment configurations wisely
5. Maintain deployment history

## Common Issues and Solutions

### 1. Deployment Failures
1. Check instance health
2. Verify IAM permissions
3. Review script logs
4. Check network connectivity
5. Validate AppSpec file

### 2. Agent Issues
1. Agent status check
2. Service restart
3. Version compatibility
4. Resource constraints
5. Network configuration

## Compliance and Auditing

### 1. AWS CloudTrail Integration
- API activity logging
- User tracking
- Resource changes
- Security analysis

### 2. Tagging Strategy
```json
{
    "Tags": [
        {
            "Key": "Environment",
            "Value": "Production"
        },
        {
            "Key": "Application",
            "Value": "MyApp"
        },
        {
            "Key": "Owner",
            "Value": "TeamA"
        }
    ]
}
```
