---
name: serverless-architecture
description: Complete serverless architecture implementation with AWS Lambda, API Gateway, DynamoDB, S3, CloudFormation, and best practices for scalable, cost-effective serverless applications.
tags: [serverless, aws, lambda, api-gateway, dynamodb, cloudformation]
version: 1.0.0
author: SoftwDocs
---

# Serverless Architecture

## Overview

A comprehensive skill for building serverless applications on AWS. Covers Lambda functions, API Gateway, DynamoDB, S3, CloudFormation, event-driven patterns, and best practices for scalable, cost-effective serverless architectures.

## When to Use This Skill

- Building serverless APIs
- Implementing event-driven architectures
- Creating cost-effective applications
- Building microservices without servers
- Implementing auto-scaling applications
- Reducing operational overhead

## Core Technologies

### Compute
- AWS Lambda for serverless functions
- Lambda Layers for shared code
- Lambda@Edge for edge computing

### Storage
- DynamoDB for NoSQL database
- S3 for object storage
- RDS Proxy for relational databases

### Integration
- API Gateway for REST/WebSocket APIs
- EventBridge for event routing
- SNS/SQS for messaging

### Infrastructure
- CloudFormation for IaC
- SAM for simplified deployment
- CDK for modern IaC

## Implementation Patterns

### Pattern 1: Lambda Function with API Gateway

Serverless REST API with Lambda:

```typescript
// lambda/handler.ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';

export const handler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  try {
    const { httpMethod, path, body } = event;
    
    if (httpMethod === 'GET' && path === '/users') {
      return {
        statusCode: 200,
        body: JSON.stringify({ users: await getUsers() }),
      };
    }
    
    if (httpMethod === 'POST' && path === '/users') {
      const userData = JSON.parse(body || '{}');
      const user = await createUser(userData);
      return {
        statusCode: 201,
        body: JSON.stringify(user),
      };
    }
    
    return {
      statusCode: 404,
      body: JSON.stringify({ error: 'Not found' }),
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' }),
    };
  }
};

// lambda/users.ts
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import {
  DynamoDBDocumentClient,
  PutCommand,
  GetCommand,
  ScanCommand,
} from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);
const TABLE_NAME = process.env.USERS_TABLE!;

export async function createUser(userData: any) {
  const command = new PutCommand({
    TableName: TABLE_NAME,
    Item: {
      id: crypto.randomUUID(),
      ...userData,
      createdAt: new Date().toISOString(),
    },
  });
  
  await docClient.send(command);
  return userData;
}

export async function getUser(id: string) {
  const command = new GetCommand({
    TableName: TABLE_NAME,
    Key: { id },
  });
  
  const response = await docClient.send(command);
  return response.Item;
}

export async function getUsers() {
  const command = new ScanCommand({
    TableName: TABLE_NAME,
  });
  
  const response = await docClient.send(command);
  return response.Items;
}
```

### Pattern 2: CloudFormation Template

Infrastructure as Code with CloudFormation:

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Serverless API with Lambda and API Gateway

Parameters:
  Stage:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - staging
      - prod

Resources:
  # DynamoDB Table
  UsersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub ${Stage}-users
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST
      StreamSpecification:
        StreamViewType: NEW_AND_OLD_IMAGES

  # Lambda Execution Role
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: DynamoDBAccess
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - dynamodb:PutItem
                  - dynamodb:GetItem
                  - dynamodb:Scan
                  - dynamodb:Query
                Resource: !GetAtt UsersTable.Arn

  # Lambda Function
  UsersLambda:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: !Sub ${Stage}-users-api
      Runtime: nodejs20.x
      Handler: index.handler
      Code:
        S3Bucket: !Ref CodeBucket
        S3Key: lambda.zip
      Role: !GetAtt LambdaExecutionRole.Arn
      Environment:
        Variables:
          USERS_TABLE: !Ref UsersTable
          STAGE: !Ref Stage
      Timeout: 30
      MemorySize: 256

  # API Gateway
  ApiGateway:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: !Sub ${Stage}-users-api
      Description: Serverless Users API

  # API Resource
  UsersResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref ApiGateway
      ParentId: !GetAtt ApiGateway.RootResourceId
      PathPart: users

  # GET Method
  GetUsersMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref ApiGateway
      ResourceId: !Ref UsersResource
      HttpMethod: GET
      AuthorizationType: NONE
      Integration:
        Type: AWS_PROXY
        IntegrationHttpMethod: POST
        Uri: !Sub arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${UsersLambda.Arn}/invocations

  # POST Method
  CreateUserMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref ApiGateway
      ResourceId: !Ref UsersResource
      HttpMethod: POST
      AuthorizationType: NONE
      Integration:
        Type: AWS_PROXY
        IntegrationHttpMethod: POST
        Uri: !Sub arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${UsersLambda.Arn}/invocations

  # Lambda Permission for API Gateway
  LambdaApiPermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref UsersLambda
      Action: lambda:InvokeFunction
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${ApiGateway}/*/*/*

  # API Deployment
  ApiDeployment:
    Type: AWS::ApiGateway::Deployment
    DependsOn:
      - GetUsersMethod
      - CreateUserMethod
    Properties:
      RestApiId: !Ref ApiGateway

  # API Stage
  ApiStage:
    Type: AWS::ApiGateway::Stage
    Properties:
      RestApiId: !Ref ApiGateway
      DeploymentId: !Ref ApiDeployment
      StageName: !Ref Stage

Outputs:
  ApiUrl:
    Description: API Gateway URL
    Value: !Sub https://${ApiGateway}.execute-api.${AWS::Region}.amazonaws.com/${Stage}
```

### Pattern 3: SAM Template

Simplified deployment with AWS SAM:

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Serverless API with SAM

Globals:
  Function:
    Runtime: nodejs20.x
    Timeout: 30
    MemorySize: 256
    Environment:
      Variables:
        STAGE: !Ref Stage

Parameters:
  Stage:
    Type: String
    Default: dev

Resources:
  UsersTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      TableName: !Sub ${Stage}-users
      PrimaryKey:
        Name: id
        Type: String

  GetUsersFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub ${Stage}-get-users
      CodeUri: ./src
      Handler: users.getUsers
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref UsersTable
      Environment:
        Variables:
          USERS_TABLE: !Ref UsersTable
      Events:
        GetUsers:
          Type: Api
          Properties:
            Path: /users
            Method: get
            RestApiId: !Ref UsersApi

  CreateUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub ${Stage}-create-user
      CodeUri: ./src
      Handler: users.createUser
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref UsersTable
      Environment:
        Variables:
          USERS_TABLE: !Ref UsersTable
      Events:
        CreateUser:
          Type: Api
          Properties:
            Path: /users
            Method: post
            RestApiId: !Ref UsersApi

  UsersApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: !Ref Stage
      Cors:
        AllowMethods: "'GET,POST,OPTIONS'"
        AllowHeaders: "'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'"
        AllowOrigin: "'*'"
```

### Pattern 4: Event-Driven Lambda

Lambda triggered by DynamoDB streams:

```typescript
// lambda/stream-handler.ts
import {
  DynamoDBStreamEvent,
  DynamoDBStreamRecord,
} from 'aws-lambda';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import {
  DynamoDBDocumentClient,
  PutCommand,
} from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (
  event: DynamoDBStreamEvent
): Promise<void> => {
  for (const record of event.Records) {
    await processRecord(record);
  }
};

async function processRecord(record: DynamoDBStreamRecord) {
  if (record.eventName === 'INSERT') {
    const newUser = record.dynamodb?.NewImage;
    if (newUser) {
      await sendWelcomeEmail(newUser);
      await createUserProfile(newUser);
    }
  }
  
  if (record.eventName === 'MODIFY') {
    const oldUser = record.dynamodb?.OldImage;
    const newUser = record.dynamodb?.NewImage;
    
    if (oldUser && newUser) {
      await handleUserUpdate(oldUser, newUser);
    }
  }
  
  if (record.eventName === 'REMOVE') {
    const deletedUser = record.dynamodb?.OldImage;
    if (deletedUser) {
      await cleanupUserData(deletedUser);
    }
  }
}

async function sendWelcomeEmail(user: any) {
  // Send welcome email using SES
  console.log('Sending welcome email to:', user.email.S);
}

async function createUserProfile(user: any) {
  // Create user profile in another table
  const command = new PutCommand({
    TableName: process.env.PROFILES_TABLE,
    Item: {
      userId: user.id.S,
      createdAt: new Date().toISOString(),
      settings: {},
    },
  });
  
  await docClient.send(command);
}
```

### Pattern 5: S3 Event Trigger

Lambda triggered by S3 events:

```typescript
// lambda/s3-handler.ts
import { S3Event } from 'aws-lambda';
import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';

const s3Client = new S3Client({});

export const handler = async (event: S3Event): Promise<void> => {
  for (const record of event.Records) {
    await processS3Object(record);
  }
};

async function processS3Object(record: any) {
  const bucket = record.s3.bucket.name;
  const key = record.s3.object.key;
  
  // Get the object
  const command = new GetObjectCommand({
    Bucket: bucket,
    Key: key,
  });
  
  const response = await s3Client.send(command);
  const content = await response.Body?.transformToString();
  
  // Process the file
  await processFile(content, key);
  
  // Generate thumbnail
  await generateThumbnail(bucket, key);
}

async function processFile(content: string, key: string) {
  // Process file content
  console.log(`Processing file: ${key}`);
}

async function generateThumbnail(bucket: string, key: string) {
  // Generate thumbnail using ImageMagick
  console.log(`Generating thumbnail for: ${key}`);
}
```

## Best Practices

### ✅ Do

- Use environment variables for configuration
- Implement proper error handling
- Use Lambda Layers for shared code
- Monitor with CloudWatch
- Implement dead-letter queues
- Use appropriate memory size
- Optimize cold starts
- Implement retry logic

### ❌ Don't

- Hardcode credentials
- Ignore timeout limits
- Skip error logging
- Use synchronous operations
- Forget to clean up resources
- Ignore cold start optimization
- Skip monitoring
- Use too much memory

## Deployment

```bash
# Deploy with SAM
sam build
sam deploy --guided

# Deploy with CloudFormation
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name serverless-api \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides Stage=prod
```

## Resources

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- [CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
