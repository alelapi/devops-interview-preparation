# AWS CloudFront

## Overview

Amazon CloudFront is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally. It integrates with other AWS services and provides an easy way to distribute content to end users with low latency and high data transfer speeds.

## Core Components

### 1. Distributions

A distribution is the basic unit of CloudFront configuration that defines how content should be delivered.

#### Distribution Types
1. **Web Distribution**
   - For static and dynamic web content
   - HTTP/HTTPS delivery
   - Multiple origin support
   - Advanced cache behaviors

2. **RTMP Distribution (Legacy)**
   - For streaming media files
   - Adobe Flash multimedia content
   - Note: Being phased out in favor of modern streaming options

### 2. Origins

Origins are the source locations of your content.

#### Supported Origin Types
1. **S3 Buckets**
   - Direct integration
   - Optional Origin Access Identity (OAI)
   - Origin Access Control (OAC)

2. **Custom Origins**
   - Application Load Balancers
   - EC2 instances
   - External web servers
   - Other HTTP endpoints

3. **Origin Groups**
   - Primary and secondary origins
   - Automatic failover
   - High availability

### 3. Cache Behaviors

Define how CloudFront handles requests for different URL patterns.

#### Key Features
- Path patterns
- Viewer protocol policy
- Allowed HTTP methods
- Cache and origin request settings
- Function associations
- Compression settings

## Features and Capabilities

### 1. Security Features

#### SSL/TLS Support
- Default CloudFront certificates
- Custom SSL certificates
- SNI support
- Dedicated IP custom SSL (additional cost)

#### Access Control
1. **Signed URLs**
   - Time-based expiration
   - IP address restrictions
   - Custom policy support

2. **Signed Cookies**
   - Multiple file access
   - Website access control
   - Session-based authentication

3. **Origin Access Identity (OAI)**
   - S3 bucket protection
   - Private content access
   - IAM integration

4. **Web Application Firewall (WAF)**
   - Rule-based filtering
   - DDoS protection
   - IP blocking

### 2. Performance Optimization

#### Caching Strategies
1. **Cache Control**
   - TTL settings
   - Cache invalidation
   - Cache behaviors
   - Query string parameters

2. **Origin Shield**
   - Additional caching layer
   - Reduced origin load
   - Improved cache hit ratio

3. **Compression**
   - Automatic compression
   - Supported formats
   - Compression thresholds

#### Edge Locations
- Global network presence
- Regional edge caches
- Automatic routing
- Load balancing

### 3. Function Integration

#### CloudFront Functions
```javascript
function handler(event) {
    // Request manipulation
    var request = event.request;
    
    // Add custom header
    request.headers['x-custom-header'] = {value: 'custom-value'};
    
    return request;
}
```

#### Lambda@Edge
```javascript
exports.handler = async (event) => {
    const request = event.Records[0].cf.request;
    
    // Modify request or generate response
    request.headers['x-custom-header'] = [{
        key: 'X-Custom-Header',
        value: 'custom-value'
    }];
    
    return request;
};
```

## Signers and Private Content

### 1. Trusted Key Groups

#### Configuration
```json
{
    "TrustedKeyGroups": {
        "Enabled": true,
        "Quantity": 1,
        "Items": ["key-group-id"]
    }
}
```

#### Key Group Management
1. **Creating Key Groups**
   - Generate public/private key pairs
   - Add public keys to CloudFront
   - Create key groups
   - Associate key groups with distributions

2. **Key Rotation**
   - Regular key rotation schedule
   - Multiple active keys support
   - Graceful transition period

### 2. CloudFront Key Pairs

#### Legacy Key Pairs (AWS Account)
- Created in AWS root account
- Maximum of two active key pairs
- Being phased out in favor of key groups
- Limited to account-wide usage

#### Key Management
```bash
# Generate RSA key pair
openssl genrsa -out private_key.pem 2048
openssl rsa -pubout -in private_key.pem -out public_key.pem
```

### 3. Signed URL Implementation

#### Using AWS SDK
```javascript
const AWS = require('aws-sdk');
const fs = require('fs');

function createSignedUrl(privateKeyPath, keyPairId, resourceUrl, expiresIn) {
    const cloudfront = new AWS.CloudFront.Signer(keyPairId, privateKeyPath);
    
    const signedUrl = cloudfront.getSignedUrl({
        url: resourceUrl,
        expires: Math.floor((Date.now() + expiresIn) / 1000)
    });
    
    return signedUrl;
}
```

### 4. Signed Cookies Implementation

#### Setting Cookies
```javascript
function setSignedCookies(policy, signature, keyPairId, domain) {
    const cookies = {
        'CloudFront-Policy': policy,
        'CloudFront-Signature': signature,
        'CloudFront-Key-Pair-Id': keyPairId
    };
    
    Object.entries(cookies).forEach(([name, value]) => {
        setCookie(name, value, {
            domain: domain,
            path: '/',
            secure: true,
            httpOnly: true
        });
    });
}
```

## Cache Configuration

### 1. Cache Behavior Settings

```json
{
    "PathPattern": "*.jpg",
    "TargetOriginId": "MyS3Origin",
    "ViewerProtocolPolicy": "redirect-to-https",
    "AllowedMethods": {
        "Quantity": 2,
        "Items": ["GET", "HEAD"]
    },
    "DefaultTTL": 86400,
    "MinTTL": 0,
    "MaxTTL": 31536000,
    "Compress": true
}
```

### 2. Cache Keys

#### Components
- URL path
- Query strings
- Headers
- Cookies
- Custom parameters

#### Optimization Strategies
1. **Query String Handling**
   - Forward all
   - Forward selected
   - None

2. **Header Forwarding**
   - Whitelist headers
   - Default headers
   - None

3. **Cookie Management**
   - Forward all
   - Forward selected
   - None

## Monitoring and Optimization

### 1. CloudWatch Integration

#### Metrics
- Request count
- Bytes transferred
- Error rates
- Cache statistics
- Origin latency

#### Alarms
- Performance thresholds
- Error thresholds
- Cost thresholds

### 2. Access Logs

#### Configuration
```json
{
    "Bucket": "my-logs-bucket",
    "Prefix": "cloudfront-logs/",
    "Enabled": true,
    "IncludeCookies": false
}
```

## Best Practices

### 1. Performance
1. Always enable compression
2. Use Origin Shield for multi-region deployments
3. Optimize cache key settings
4. Configure appropriate TTLs

### 2. Security
1. Use HTTPS for all content
2. Implement WAF rules
3. Use Origin Access Identity
4. Regular security updates
5. Implement proper signer key rotation
6. Secure storage of private keys
7. Monitor key usage

### 3. Reliability
1. Use origin failover
2. Monitor performance metrics
3. Set up alerting
4. Regular testing

## Troubleshooting

### 1. Common Issues
1. Cache misses
2. Origin errors
3. SSL/TLS issues
4. Access denied errors
5. Signature validation failures
6. Invalid key pairs
7. Policy violations

### 2. Resolution Steps
1. Check configuration
2. Verify origin health
3. Review error logs
4. Test with curl
5. Verify signature validity
6. Check policy expiration
7. Validate key pair status

## Integration Examples

### 1. S3 Integration
```terraform
resource "aws_cloudfront_distribution" "s3_distribution" {
  origin {
    domain_name = aws_s3_bucket.website.bucket_regional_domain_name
    origin_id   = "S3Origin"
    
    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.oai.cloudfront_access_identity_path
    }
  }
  
  enabled = true
  default_root_object = "index.html"
  
  default_cache_behavior {
    target_origin_id = "S3Origin"
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    
    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
}
```

### 2. Custom Origin Integration
```terraform
resource "aws_cloudfront_distribution" "custom_distribution" {
  origin {
    domain_name = "api.example.com"
    origin_id   = "CustomOrigin"
    
    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }
  
  enabled = true
  
  default_cache_behavior {
    target_origin_id = "CustomOrigin"
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    
    forwarded_values {
      query_string = true
      cookies {
        forward = "all"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
  }
}
```
