# Serverless Application Model (SAM)

## Overview
AWS SAM is a framework for developing and deploying serverless applications, providing a simplified approach to creating serverless infrastructure.

## Key Characteristics
* Configuration-as-Code using YAML
* Generates complex CloudFormation templates from simple SAM templates
* Full CloudFormation compatibility
* Supports comprehensive resource configuration

## Core Components

### Template Structure
* Uses special transform header: `Transform: 'AWS::Serverless-2016-10-31'`
* Supports key serverless resources:
  * `AWS::Serverless::Function`
  * `AWS::Serverless::Api`
  * `AWS::Serverless::SimpleTable`

### Deployment Process
1. Build: `sam build`
   * Prepares application locally
2. Package: `sam package`
   * Transforms and uploads application code to S3
3. Deploy: `sam deploy`
   * Creates/updates CloudFormation stack
   * Provisions serverless resources

## Advanced Features

### SAM Accelerate
* Reduces deployment latency
* Synchronization options:
  * `sam sync`: Full resource synchronization
  * `sam sync --code`: Quick code updates
  * `sam sync --watch`: Automatic file change detection

### Local Development Capabilities
* `sam local start-lambda`: Local Lambda endpoint
* `sam local invoke`: Invoke Lambda functions locally
* `sam local start-api`: Local API Gateway simulation
* `sam local generate-event`: Generate sample event payloads

### CodeDeploy Integration
* Native traffic shifting for Lambda functions
* Deployment strategies:
  * Canary
  * Linear
  * All At Once
* Support for:
  * Pre/Post traffic hooks
  * Automated rollback
  * CloudWatch Alarm triggers

### Policy Templates
Predefined IAM permission templates for Lambda functions:
* `S3ReadPolicy`: S3 read permissions
* `SQSPollerPolicy`: SQS queue polling
* `DynamoDBCrudPolicy`: Database CRUD operations

## Multi-Environment Support
* Configuration via `samconfig.toml`
* Environment-specific deployments
* Example: `sam deploy --config-env dev`

## Benefits
* Simplified serverless development
* Rapid local testing
* Consistent deployment patterns
* Reduced infrastructure configuration complexity
* Seamless AWS service integration
