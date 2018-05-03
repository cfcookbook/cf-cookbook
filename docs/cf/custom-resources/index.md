# Minimal Inline Custom Resource

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Resources:
  MyCustomResourceLambdaRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - lambda.amazonaws.com
          Action:
          - sts:AssumeRole
      Path: "/"
      Policies:
      - PolicyName: root
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Action:
            - logs:*
            Resource: arn:aws:logs:*:*:*

  MyCustomResourceLambda:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.handler
      Timeout: 30
      Runtime: python2.7
      Role: !GetAtt MyCustomResourceLambdaRole.Arn

      MemorySize: 256
      Code:
        ZipFile: |
          import json
          from botocore.vendored import requests

          def handler(event, context):
            print(json.dumps(event, indent=2))
            phys_id = "{}-{}".format(event['LogicalResourceId'], event['ServiceToken'][-12:])
            payload = {
              'Status': "SUCCESS",
              'PhysicalResourceId': phys_id,
              'StackId': event['StackId'],
              'RequestId': event['RequestId'],
              'LogicalResourceId': event['LogicalResourceId'],
              'Data': {}
            }
            # This is what returns our status
            r = requests.put(event['ResponseURL'], data=json.dumps(payload), timeout=30)
            return r.status_code

  MyCustomResource:
    Type: Custom::MyCustomResource
    Properties:
      ServiceToken: !GetAtt MyCustomResourceLambda.Arn
      ConfigItems:
        - whatever
        - something else
      Greeting: !Sub "Hello ${AWS::Region} World"
```
