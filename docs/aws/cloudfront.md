# CloudFront

## Service Definition
AWS CloudFront is a Content Delivery Network (CDN) service that dramatically improves content delivery performance. With 216 global points of presence (edge locations), it offers enhanced read performance by caching content strategically worldwide.

## Key Security and Performance Features
CloudFront provides robust DDoS protection through integration with AWS Shield and Web Application Firewall. Its global infrastructure ensures content is cached close to end-users, significantly improving access speeds and user experience.

## Origin Types
CloudFront supports multiple origin types for content delivery:

### S3 Bucket Origins
- Enables file distribution and edge caching
- Implements enhanced security through Origin Access Control (OAC)
- Replaces older Origin Access Identity (OAI) mechanism
- Can be used as an ingress point for S3 uploads

### Custom Origins
- Supports HTTP-based origins including:
  - Application Load Balancers
  - EC2 instances
  - Static S3 websites
  - Any HTTP backend system

## Caching Mechanism

### Cache Key
The cache key is a unique identifier for each cached object, typically comprising the hostname and URL's resource portion. CloudFront allows sophisticated cache key customization by incorporating:
- HTTP headers
- Cookies
- Query strings
- Device information
- User location

### Cache Policies
CloudFront offers extensive cache policy configurations:

#### HTTP Headers
- None: Excludes headers from cache key
- Whitelist: Includes specific headers in cache key

#### Query Strings
- None: Excludes query strings from cache key
- Whitelist: Includes specific query strings
- Include All-Except: Includes all query strings except specified ones
- All: Includes all query strings (lowest caching performance)

#### Origin Request Policy
Enables including values in origin requests without duplicating cached content, supporting:
- HTTP headers configuration
- Cookie management
- Query string handling
- Custom header addition

## Advanced Features

### Cache Invalidation
When backend origins update, CloudFront allows forced cache refresh through:
- Full cache invalidation
- Partial path invalidation (e.g., /images/*)

### Cache Behaviors
Configures distinct settings for specific URL path patterns, enabling:
- Different origin routing based on content type
- Customized caching for specific file types
- Prioritized processing of cache behaviors

### Geographical Restrictions
Implements content access control through:
- Allowlist: Permit access from specific countries
- Blocklist: Prevent access from specified countries
- Uses third-party Geo-IP database for determination

### Signed URL and Cookies
Provides secure content distribution mechanisms:
- URL/cookie expiration control
- IP range access restrictions
- Trusted signer management
- Supports individual file (Signed URL) and multiple file (Signed Cookies) access

### Field-Level Encryption
Offers additional security layer by:
- Encrypting sensitive information at edge locations
- Supporting up to 10 encrypted fields in POST requests
- Utilizing asymmetric encryption

### Real-Time Logging
Streams CloudFront requests to Kinesis Data Streams, enabling:
- Real-time performance monitoring
- Configurable sampling rates
- Selective field and path pattern tracking

## Origin Groups
Enhances availability through:
- Primary and secondary origin configuration
- Automatic failover mechanisms

## Conclusion
AWS CloudFront provides a comprehensive, flexible content delivery solution with robust security, performance optimization, and global reach.
