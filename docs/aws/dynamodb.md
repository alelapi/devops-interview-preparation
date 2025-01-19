# Amazon DynamoDB Documentation

## Overview
Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. It offers encryption at rest, automated backup and restore, and in-memory caching.

## Core Concepts

### Tables, Items, and Attributes
- **Tables**: Container for all items
- **Items**: Group of attributes (similar to rows)
- **Attributes**: Fundamental data elements (similar to columns)
- **Primary Key**: Unique identifier for each item
  - Simple Primary Key (Partition Key only)
  - Composite Primary Key (Partition Key + Sort Key)

### Data Types
1. **Scalar Types**
   - Number
   - String
   - Binary
   - Boolean
   - Null

2. **Document Types**
   - List
   - Map

3. **Set Types**
   - Number Set
   - String Set
   - Binary Set

## Key Features

### Performance
- Single-digit millisecond latency
- Auto-scaling capabilities
- Provisioned or On-demand capacity modes
- Global tables for multi-region deployment

### Durability and Availability
- Data replicated across multiple AZs
- Point-in-time recovery
- On-demand backup and restore
- Automated continuous backups

### Security
- Encryption at rest using AWS KMS
- Encryption in transit using TLS
- Fine-grained access control with IAM
- VPC endpoints support

## Implementation Examples

### Table Creation (AWS SDK - Python)
```python
import boto3

dynamodb = boto3.resource('dynamodb')

table = dynamodb.create_table(
    TableName='Users',
    KeySchema=[
        {
            'AttributeName': 'user_id',
            'KeyType': 'HASH'  # Partition key
        },
        {
            'AttributeName': 'timestamp',
            'KeyType': 'RANGE'  # Sort key
        }
    ],
    AttributeDefinitions=[
        {
            'AttributeName': 'user_id',
            'AttributeType': 'S'
        },
        {
            'AttributeName': 'timestamp',
            'AttributeType': 'N'
        },
    ],
    ProvisionedThroughput={
        'ReadCapacityUnits': 5,
        'WriteCapacityUnits': 5
    }
)

table.wait_until_exists()
```

### Basic CRUD Operations (Python)
```python
# Create/Update Item
def put_item(table_name, item):
    table = dynamodb.Table(table_name)
    response = table.put_item(Item=item)
    return response

# Read Item
def get_item(table_name, key):
    table = dynamodb.Table(table_name)
    response = table.get_item(Key=key)
    return response.get('Item')

# Update Item
def update_item(table_name, key, update_expression, expression_values):
    table = dynamodb.Table(table_name)
    response = table.update_item(
        Key=key,
        UpdateExpression=update_expression,
        ExpressionAttributeValues=expression_values,
        ReturnValues="UPDATED_NEW"
    )
    return response

# Delete Item
def delete_item(table_name, key):
    table = dynamodb.Table(table_name)
    response = table.delete_item(Key=key)
    return response
```

### Query and Scan Operations
```python
# Query Example
def query_user_items(table_name, user_id):
    table = dynamodb.Table(table_name)
    response = table.query(
        KeyConditionExpression='user_id = :uid',
        ExpressionAttributeValues={
            ':uid': user_id
        }
    )
    return response['Items']

# Scan Example with Filter
def scan_with_filter(table_name, filter_expression):
    table = dynamodb.Table(table_name)
    response = table.scan(
        FilterExpression=filter_expression
    )
    return response['Items']
```

## Best Practices

### Table Design
1. **Choose Appropriate Primary Keys**
   - Ensure even distribution of data
   - Avoid hot partitions
   - Consider access patterns

2. **Use Secondary Indexes Wisely**
   - Global Secondary Indexes (GSI)
   - Local Secondary Indexes (LSI)
   - Limit number of indexes to minimize costs

3. **Data Modeling**
   - Denormalize data for query efficiency
   - Use composite sort keys for hierarchical data
   - Consider item collections

### Performance Optimization
1. **Capacity Planning**
   - Choose appropriate capacity mode
   - Use auto-scaling
   - Monitor capacity metrics

2. **Query Optimization**
   - Use query over scan
   - Implement pagination
   - Use parallel scans when appropriate

3. **Caching Strategy**
   - Implement DAX for caching
   - Use appropriate TTL settings
   - Monitor cache hit ratios

## Common Design Patterns

### One-to-Many Relationships
```json
{
    "OrderID": "12345",
    "CustomerID": "C101",
    "OrderItems": [
        {
            "ProductID": "P1",
            "Quantity": 2
        },
        {
            "ProductID": "P2",
            "Quantity": 1
        }
    ]
}
```

### Many-to-Many Relationships
```json
{
    "GSI1PK": "USER#123",
    "GSI1SK": "GROUP#456",
    "Type": "UserGroup",
    "UserID": "123",
    "GroupID": "456",
    "JoinDate": "2023-01-01"
}
```

## Monitoring and Operations

### CloudWatch Metrics to Monitor
- ConsumedReadCapacityUnits
- ConsumedWriteCapacityUnits
- ThrottledRequests
- SystemErrors
- UserErrors

### Common Operations Tasks
1. **Backup and Restore**
   - On-demand backups
   - Point-in-time recovery
   - Cross-region backup

2. **Scaling**
   - Update capacity
   - Enable auto-scaling
   - Monitor throttling

3. **Maintenance**
   - Update indexes
   - Cleanup expired items
   - Optimize tables

## Cost Optimization

### Cost Factors
1. **Storage**
   - Charged per GB stored
   - Consider data lifecycle

2. **Throughput**
   - Read/Write Capacity Units
   - On-demand vs Provisioned

3. **Features**
   - Backup storage
   - Global Tables replication
   - DAX nodes

### Optimization Strategies
1. Use appropriate capacity mode
2. Implement TTL for data expiration
3. Compress large attributes
4. Use batch operations
5. Monitor and adjust capacity

## Limits and Quotas
- Item size: 400KB
- Partition key length: 2048 bytes
- Sort key length: 1024 bytes
- Tables per account: 256 (default)
- Secondary indexes per table: 20 (5 LSI, 15 GSI)
- Projected attributes per index: 100

## Error Handling and Troubleshooting

### Common Errors
1. ProvisionedThroughputExceededException
2. ResourceNotFoundException
3. ConditionalCheckFailedException
4. ValidationException
