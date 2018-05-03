# Basic Dynamo Tables

These examples generally have very low `ProvisionedThroughput` set, as well as `SSEEnabled` (server side encryption).


## Hash and Range Key

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Resources:
  Songs:
    Type: "AWS::DynamoDB::Table"
    Properties:
      AttributeDefinitions:
        - AttributeName: "ArtistName"
          AttributeType: "S"
        - AttributeName: "SongTitle"
          AttributeType: "S"
      KeySchema:
        - AttributeName: "ArtistName"
          KeyType: "HASH"
        - AttributeName: "SongTitle"
          KeyType: "RANGE"
      SSESpecification:
        SSEEnabled: True
      ProvisionedThroughput:
        ReadCapacityUnits: 1
        WriteCapacityUnits: 1
```
## Secondary index

Note that each [Global Secondary Index](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html) is basically treated as a new/separate table in terms of pricing and throughput provisioning.

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Resources:
  Songs:
    Type: "AWS::DynamoDB::Table"
    Properties:
      AttributeDefinitions:
        - AttributeName: "ArtistName"
          AttributeType: "S"
        - AttributeName: "SongTitle"
          AttributeType: "S"
      Songs:
        - AttributeName: "ArtistName"
          KeyType: "HASH"
        - AttributeName: "SongTitle"
          KeyType: "RANGE"
      ProvisionedThroughput:
        ReadCapacityUnits: 1
        WriteCapacityUnits: 1
      SSESpecification:
        SSEEnabled: True
      GlobalSecondaryIndexes:
        - IndexName: LookupByTitle
          KeySchema:
            - AttributeName: "SongTitle"
              KeyType: "HASH"
            - AttributeName: "ArtistName"
              KeyType: "RANGE" 
          Projection:
            ProjectionType: ALL
          ProvisionedThroughput:
            ReadCapacityUnits: 1
            WriteCapacityUnits: 1
```

## Features Not Supported by CloudFormation

- [Point in Time Recovery](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery_Howitworks.html)
    - [CLI instructions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.Tutorial.html)
    - Note that a restore creates a new table with the restored data.
