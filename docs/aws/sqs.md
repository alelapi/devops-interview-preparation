# Amazon Simple Queue Service (SQS)

## Overview
Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables decoupling and scaling of distributed systems and applications. It provides a secure, durable, and available hosted queue for storing messages while they travel between applications or microservices.

## Key Features

### Queue Types
1. Standard Queues
   - Unlimited throughput
   - At-least-once delivery
   - Best-effort ordering
   - Ideal for high-throughput applications where order is not critical

2. FIFO Queues
   - First-In-First-Out delivery guarantee
   - Exactly-once processing
   - Limited to 300 transactions per second (TPS)
   - Perfect for applications requiring strict ordering

### Message Properties
- Size: Up to 256KB of text in any format
- Retention Period: 1 minute to 14 days (default 4 days)
- Short Polling vs Long Polling options
- Visibility Timeout: 0 seconds to 12 hours

## Core Concepts

### Message Lifecycle
1. Producer sends message to SQS queue
2. Message is redundantly distributed across SQS servers
3. Consumer receives and processes the message
4. Message is deleted from the queue by the consumer

### Visibility Timeout
- Period during which SQS prevents other consumers from receiving and processing the same message
- Default: 30 seconds
- Maximum: 12 hours
- Can be changed via ChangeMessageVisibility API

### Dead Letter Queues (DLQ)
- Isolate messages that can't be processed successfully
- Useful for debugging and monitoring
- Messages moved to DLQ after exceeding MaxReceiveCount
- Separate queue that must be configured manually

## Security Features

### Access Control
- IAM policies for queue-level permissions
- SQS Access Policy for cross-account access
- Temporary security credentials via AWS STS

### Encryption
- Server-Side Encryption (SSE) using AWS KMS
- Transport layer security (TLS) for in-transit encryption
- Optional client-side encryption

## Best Practices

### Message Processing
1. Implement idempotent processing
2. Handle message deduplication in FIFO queues
3. Set appropriate visibility timeout
4. Implement proper error handling
5. Use batch operations when possible

### Performance Optimization
1. Use long polling to reduce API calls
2. Implement concurrent processing
3. Handle partial batch failures
4. Monitor queue metrics
5. Scale consumers based on queue depth

### Cost Optimization
1. Use batch operations (SendMessageBatch, DeleteMessageBatch)
2. Delete messages promptly after processing
3. Consider message size impact
4. Monitor API usage

## Integration Patterns

### Common Use Cases
1. Application decoupling
2. Workload buffering
3. Request batching
4. Fan-out architecture
5. Auto-scaling trigger
6. Load leveling

### Service Integration
- AWS Lambda
- Amazon EC2
- Amazon ECS
- AWS Step Functions
- Amazon EventBridge

## Monitoring and Troubleshooting

### CloudWatch Metrics
- ApproximateNumberOfMessages
- ApproximateAgeOfOldestMessage
- NumberOfMessagesDeleted
- NumberOfMessagesReceived
- SentMessageSize

### Common Issues
1. Messages not being deleted
2. Visibility timeout too short/long
3. Dead letter queue configuration
4. IAM permission issues
5. Queue depth management

## Pricing Considerations
- Pay for use with no minimum fees
- Charged per 1 million requests
- Data transfer charges apply
- Additional charges for FIFO queues
- Free tier available for new AWS accounts

## Limits and Quotas
- Message size: 256KB maximum
- Message retention: 14 days maximum
- Queue names: 80 characters maximum
- Queues per account: 1,000 (default)
- In-flight messages: 120,000 (standard) or 20,000 (FIFO)
