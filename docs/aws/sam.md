# AWS SAM (Serverless Application Model) Documentation

## Overview
The AWS Serverless Application Model (SAM) is an open-source framework for building serverless applications. It extends AWS CloudFormation to provide a simplified way to define serverless applications. It provides shorthand syntax to express functions, APIs, databases, and event source mappings. With just a few lines per resource, you can define the application you want and model it using YAML.

SAM supports the following resource types:

- AWS::Serverless::Api
- AWS::Serverless::Application
- AWS::Serverless::Function
- AWS::Serverless::HttpApi
- AWS::Serverless::LayerVersion
- AWS::Serverless::SimpleTable
- AWS::Serverless::StateMachine

## Key Components

### 1. SAM Template Structure
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: SAM Template Example

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs14.x
      CodeUri: ./src/
```

### 2. Main Resource Types

#### AWS::Serverless::Function
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Handler: app.handler
    Runtime: python3.9
    CodeUri: ./
    MemorySize: 128
    Timeout: 3
    Environment:
      Variables:
        TABLE_NAME: !Ref MyTable
    Events:
      ApiEvent:
        Type: Api
        Properties:
          Path: /hello
          Method: get
```

#### AWS::Serverless::Api
```yaml
MyApi:
  Type: AWS::Serverless::Api
  Properties:
    StageName: prod
    Cors:
      AllowMethods: "'GET,POST,OPTIONS'"
      AllowHeaders: "'content-type'"
      AllowOrigin: "'*'"
```

#### AWS::Serverless::LayerVersion
```yaml
MyLayer:
  Type: AWS::Serverless::LayerVersion
  Properties:
    LayerName: my-layer
    Description: My layer description
    ContentUri: ./layer/
    CompatibleRuntimes:
      - python3.9
```

## Common Commands

### Installation
```bash
# Install AWS SAM CLI
pip install aws-sam-cli

# Verify installation
sam --version
```

### Project Management
```bash
# Create new project
sam init

# Build project
sam build

# Deploy application
sam deploy --guided

# Local testing
sam local start-api
sam local invoke "FunctionName"

# Package application
sam package \
  --template-file template.yaml \
  --output-template-file packaged.yaml \
  --s3-bucket my-bucket
```

## Local Development

### Local API Testing
```bash
# Start local API
sam local start-api

# Invoke single function
sam local invoke -e events/event.json

# Generate sample event
sam local generate-event apigateway aws-proxy
```

### Environment Variables
```yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Environment:
        Variables:
          DB_NAME: my-database
          API_KEY: !Ref ApiKey
```

## Event Source Mappings

### API Gateway
```yaml
Events:
  ApiEvent:
    Type: Api
    Properties:
      Path: /hello
      Method: get
```

### S3 Events
```yaml
Events:
  S3Event:
    Type: S3
    Properties:
      Bucket: !Ref MyBucket
      Events: s3:ObjectCreated:*
```

### DynamoDB Events
```yaml
Events:
  StreamEvent:
    Type: DynamoDB
    Properties:
      Stream: !GetAtt MyTable.StreamArn
      StartingPosition: LATEST
      BatchSize: 100
```

## Best Practices

### 1. Security
- Use IAM roles with least privilege
- Enable X-Ray for tracing
- Implement proper error handling
- Use Secrets Manager for sensitive data

### 2. Performance
- Optimize function memory allocation
- Use layers for common dependencies
- Implement caching where appropriate
- Consider cold start impact

### 3. Monitoring
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Tracing: Active
    AutoPublishAlias: live
    DeploymentPreference:
      Type: Canary10Percent5Minutes
```

### 4. Cost Optimization
- Use appropriate memory settings
- Implement proper timeouts
- Use provisioned concurrency when needed
- Monitor and adjust based on usage

## Testing and Debugging

### Unit Testing
```python
# Example test using pytest
def test_handler():
    response = app.handler(event, context)
    assert response['statusCode'] == 200
```

### SAM Local Debugging
```bash
# Debug with VS Code
sam local invoke -d 5858 MyFunction

# Generate sample events
sam local generate-event s3 put
```

## CI/CD Integration

### GitHub Actions Example
```yaml
name: SAM Deploy
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: aws-actions/setup-sam@v1
      - uses: aws-actions/configure-aws-credentials@v1
      - run: sam build
      - run: sam deploy --no-confirm-changeset
```

### AWS CodePipeline Integration
```yaml
Pipeline:
  Type: AWS::CodePipeline::Pipeline
  Properties:
    Stages:
      - Name: Source
        Actions:
          - Name: Source
            ActionTypeId:
              Category: Source
              Provider: GitHub
      - Name: Build
        Actions:
          - Name: Build
            ActionTypeId:
              Category: Build
              Provider: CodeBuild
```

## Troubleshooting

### Common Issues
1. **Deployment Failures**
   - Check IAM permissions
   - Verify S3 bucket accessibility
   - Review CloudFormation errors

2. **Function Execution Issues**
   - Check CloudWatch Logs
   - Verify environment variables
   - Test locally with SAM CLI

3. **API Gateway Issues**
   - Verify CORS settings
   - Check API Gateway logs
   - Test endpoints locally

## Advanced Features

### Custom Runtime
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Runtime: provided
    Handler: bootstrap
    CodeUri: ./custom-runtime/
```

### VPC Configuration
```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    VpcConfig:
      SecurityGroupIds:
        - sg-123456
      SubnetIds:
        - subnet-123456
```

### Step Functions Integration
```yaml
StateMachine:
  Type: AWS::Serverless::StateMachine
  Properties:
    DefinitionUri: statemachine/definition.asl.json
    Policies:
      - LambdaInvokePolicy:
          FunctionName: !Ref MyFunction
```

## Production Considerations

### 1. Deployment Strategies
- Use staged deployments
- Implement rollback mechanisms
- Consider traffic shifting
- Use aliases for version management

### 2. Monitoring and Alerting
- Set up CloudWatch Alarms
- Configure X-Ray tracing
- Implement custom metrics
- Set up logging standards

### 3. Scaling and Performance
- Configure concurrency limits
- Use provisioned concurrency
- Implement caching strategies
- Optimize cold starts
