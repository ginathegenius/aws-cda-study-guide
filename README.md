# AWS Certified Developer Associate
## Table of Contents
- [Serverless](#serverless)
  - [Lambda](#lambda)
  - [DynamoDB](#dynamodb)
  - [SQS](#sqs)
  - [S3](#s3)
- [Elastic Beanstalk](#elastic-beanstalk)
- [IAM](#iam)
- [Cognito](#cognito)
- [Security Token Service](#security-token-service)
- [ECS](#ecs)
- [CI/CD](#ci-cd)
- [EC2](#ec2)
- [Elastic Load Balancer](#elastic-load-balancer)
- [RDS](#rds)
- [Cloudformation](#cloudformation)
- [Kinesis](#kinesis)
- [Cloudfront](#cloudfront)
- [ElastiCache](#elasticache)
- [CloudWatch](#cloudwatch)
- [X-Ray](#x-ray)
- [Secrets Manager & Parameter Store](#secrets-manager-&-parameter-store)


## Serverless
- Lambda, Fargate, EventBridge, Step Functions, SQS, SNS, API Gateway, AppSync, S3, DynamoDB, RDS Proxy, Aurora Serverless

### Lambda
- If a Lambda function is connected to a VPC, it must have a route out to the internet via a NAT Gateway or NAT instance in order to connect to external services.
- You must also ensure that the relevant Security Groups and Network Access Control Lists are configured to allow the required ports.
#### Versioning
- When you create a Lambda function, there is only one version: $LATEST. 
- You can refer to the function using its Amazon Resource Name (ARN). There are two ARNs associated with this initial version, the qualified ARN which is the function ARN plus a version suffix e.g. $LATEST. Or the unqualified ARN which is the function ARN without the version suffix. 
- The function version for an unqualified function always maps to $LATEST, so you can access the latest version using either the qualified ARN with $LATEST, or the unqualified function ARN. 
- Lambda also supports creating aliases for each of your Lambda function versions. An alias is a pointer to a specific Lambda function version, aliases will not be updated automatically when a new version of the function becomes available.

### DynamoDB
- Primary key attribute must be defined as type string, number, or binary
- All DynamoDB tables are encrypted at rest using an AWS owned CMK by default. Non-encrypted DynamoDB tables are no longer supported in AWS. You have the option to pick an alternative AWS or Customer Managed KMS key if required
- A well designed Sort key allows you to retrieve groups of related items and query based on a range of values, e.g. a range of dates

#### Query vs Scan 
| Query | Scan|
|---|--|
|finds items based on primary key values|reads every item in a table or a secondary index|

**Projection Expression** - to retrieve select attributes after a Query or Scan

#### Indexes
| Local Secondary Index | Global Secondary Index|
|-----------------------|-----------------------|
| Must be created on table creation|Can be created at any time|
|Same partition key as table|Different partition key|
|Different sort key|Different sort key|

#### Provisioned Throughput 
- A read capacity unit is 4KB in size

Strongly Consistent Reads 
```
size of each item / 4 KB (round) * # of reads per second
```
Eventually Consistent Reads 
```
(size of each item / 4 KB (round) * # of reads per second) / 2
```
Write Capacity Units 
```
size of each item / 1 KB (round) * # of writes per second
```

### SQS
- A fully managed message queuing service that enables you to decouple and scale micro-services, distributed systems, and serverless applications
- Standard queues
- FIFO queues 
- SQS has a maximum message size of 256 KB
#### Short & Long Polling
By default queues use short polling.

##### Short polling
- Queries only a subset of the servers (based on a weighted random distribution) to find messages that are available to include in the response
- Sends the response right away, even if the query found no messages 
 
##### Long polling
- Queries all of the servers for messages
- Sends a response after it collects at least one available message, up to the maximum number of messages specified in the request
- Sends an empty response only if the polling wait time expires
- Reduces cost by eliminating # of empty responses and false empty responses

#### Visibility timeout

When a consumer receives and processes a message from the queue, the message remains in the queue. SQS does not automatically delete the message. When the message is received, it remains in the queue. To prevent other consumers from processing the message again, Amazon SQS set a visibility timeout, a period of time that prevents other consumers from receiving and processing the message **after it has been consumed**. The default is 30 seconds, min is 0 sec and max is 12 hr.

#### Delay Queues
- Postpone the delivery of new messages to a queue for a number of seconds
- The default (minimum) delay for a queue is 0 seconds
- The maximum is 15 minutes
- For standard queues, the per-queue delay setting is not retroactive—changing the setting doesn't affect the delay of messages already in the queue.
- For FIFO queues, the per-queue delay setting is retroactive—changing the setting affects the delay of messages already in the queue.

**The difference between the two is that, for delay queues, a message is hidden when it is first added to queue, whereas for visibility timeouts a message is hidden only after it is consumed from the queue.** 

#### Dead-Letter Queues

Allows you to set aside and isolate messages that cant be processed correctly to determine why their processing didn't succeed. Allows you to prevent data loss.

##### When to use

- **Do** use dead-letter queues with standard queues. You should always take advantage of dead-letter queues when your applications don’t depend on ordering. Dead-letter queues can help you troubleshoot incorrect message transmission operations.

- **Do** use dead-letter queues to decrease the number of messages and to reduce the possibility of exposing your system to poison-pill messages (messages that can be received but can’t be processed).

- **Don’t** use a dead-letter queue with standard queues when you want to be able to keep retrying the transmission of a message indefinitely. For example, don’t use a dead-letter queue if your program must wait for a dependent process to become active or available.

- **Don’t** use a dead-letter queue with a FIFO queue if you don’t want to break the exact order of messages or operations. For example, don’t use a dead-letter queue with instructions in an Edit Decision List (EDL) for a video editing suite, where changing the order of edits changes the context of subsequent edits.

### S3
- Files can be between 0-5TB
- The largest file you can transfer to S3 using a PUT operation is 5GB
- AWS recommends that you use **Multipart Upload** for files larger than 100MB
- **S3 Transfer Acceleration** is recommended to increase upload speeds and especially useful in cases where your bucket resides in a Region other than the one in which the file transfer was originated
- Cross-origin resource sharing (CORS) defines a way for client web applications that are loaded in one domain to interact with resources in a different domain
- S3 buckets do not directly support HTTPS with a custom domain name

#### Encryption
- If file is to be encrypted at upload time, the `x-amz-server-side-encryption` parameter will be included in request header 
- Server-Side Encryption
    - S3 Managed Keys – SSE-S3 
    - AWS Key management Services KMS, managed keys - SSE-KMS 
    - Server side encryption with customer provided keys – SSE-C 
- Client Side Encryption
    - Customer encrypts files before upload by leveraging AWS Encryption SDK

### API Gateway
- API Gateway supports RESTful APIs, however the legacy SOAP protocol, which returns results in xml format, is also supported in pass-through mode
- If your existing services return XML-style data, you can use the API Gateway to transform the output to JSON


## EC2
It is best practice to deploy the SSL certificates on the Load Balancer. This implements SSL termination on the load balancer and off-loads this task from the application, thus reducing the load on EC2 instances. Additionally, it removes the requirement of distributing the certificate to all target EC2 instances

## Elastic Load Balancer
- It is best practice to deploy the SSL certificates on the Load Balancer. This implements SSL termination on the load balancer and off-loads this task from the application, thus reducing the load on EC2 instances. Additionally, it removes the requirement of distributing the certificate to all target EC2 instances
- 



## RDS
- Security best practice would state that RDS Database instances should be deployed to a private subnet. A private subnet would only have private IP's with no direct access to the public internet. Outbound connectivity would be provided via a NAT gateway. Thus, a route table entry with destination 0.0.0.0/0 and target is a suitable choice for deploying RDS instances as it is characterized as a private subnet

## Elastic Beanstalk
### Supported Platforms
Programming languages (Go, Java, Node.js, PHP, Python, Ruby), application servers (Tomcat, Passenger, Puma), and Docker containers
### Deployment Options
 <table>
  <tr>
    <th>All at once</th>
    <td><ul><li>The quickest deployment method</li>
    <li>Application might be unavailable to users (or have low availability) for a short time</li>
    <li>If update fails, you need to rollback the changes by re-deploying the original version to all your instances  </li>
    <li>All instances are out of service while deployment takes place </li><li>An all at once is suitable if you can accept a short loss of service, and if quick deployments are important to you</li>
    </ul></td>
  </tr>
  <tr>
    <th >Rolling</th>
    <td><ul><li> Deploys new version in batches </li>
    <li>Deploys new version in batches </li>
    <li>Reduced capacity during deployment  </li><li>Rolling is suitable if you can't accept any period of completely lost service. With this method, your application is deployed to your environment one batch of instances at a time</li>
    </ul></td>
  </tr>
    <th>Rolling with additional batch</th>
    <td><ul><li>Deploys the new version in batches  </li>
    <li>Maintains full capacity during the deployment process </li>
    <li>Reduced capacity during deployment  </li>
    </ul></td>

  </tr>
    <th >Immutable</th>
    <td><ul>
    <li>Deploys new version to a fresh group of instances in their own autoscaling group  </li>
    <li>Maintains full capacity during the deployment process </li>
    <li>Impact of failed update is far less, the roll back process requires only terminating the new auto scaling group   </li>
    <li>Preferred option for mission critical production systems  </li>
    </ul></td>
  </tr>
 </table> 



## IAM 
### Cross Account Access 
Configure cross account access by creating a role in your account which has permission to access only the resources they need. Allow the third party account to assume the role based on their account ID and unique external ID
### AWS identity federation process

## Cognito 
Amazon Cognito provides authentication, authorization, and user management for your web and mobile apps. Your users can sign in directly with a user name and password, or through a third party such as Facebook, Amazon, Google or Apple.

### User pools
User pools are user directories that provide sign-up and sign-in options for your app users. With a user pool, your users can sign in to your web or mobile app through Amazon Cognito, or federate through a third-party identity provider (IdP). Whether your users sign in directly or through a third party, all members of the user pool have a directory profile that you can access through an SDK.

### Cognito sync
Amazon Cognito Sync is an AWS service and client library that enables cross-device syncing of application-related user data. You can use it to synchronize user profile data across mobile devices and the web without requiring your own backend. The client libraries cache data locally so your app can read and write data regardless of device connectivity status. When the device is online, you can synchronize data, and if you set up push sync, notify other devices immediately that an update is available.

#### Cognito Streams
Amazon Cognito Streams gives developers control and insight into their data stored in Amazon Cognito

#### Cognito Events
Amazon Cognito Events allows you to execute an AWS Lambda function in response to important events in Amazon Cognito

### Identity pools
Identity pools enable you to grant your users access to other AWS services.



## Security Token Service
### AssumeRoleWithWebIdentity
STS AssumeRoleWithWebIdentity returns a set of temporary security credentials for users who have been authenticated in a mobile or web application with a web identity provider.

## ECS 
- AWS Fargate is a compute engine for Amazon ECS that allows you to run containers without having to manage servers or clusters
- ECS stands for Elastic Container Service: It manages running containers on your EC2 instances
- The policy must be attached to the ECS Task's execution role to allow the application running in the container access SQS

### Docker 



## CI CD 
### CodeDeploy
What permissions need to be configured to allow CodeDeploy to perform the deployment to EC2?
Create an IAM policy with an acton to allow `codecommit:GitPull` on the required repository. Attach the policy to the EC2 instance profile role.
#### Appspec file
- The AppSpec file (appspec.yml) must always be in the root or your application source directory otherwise the deployment will not work
##### Hooks
1. ApplicationStop	
2. DownloadBundle	
3. BeforeInstall 	
4. Install
5. AfterInstall 
6. ApplicationStart 	
7. ValidateService	
8. BeforeBlockTraffic
9. BlockTraffic
10. AfterBlockTraffic 
11. BeforeAllowTraffic 	
12. AllowTraffic

### CodeBuild
buildspec.yml

A buildspec is a collection of build commands and related settings, in YAML format, that CodeBuild uses to run a build

**Docker commands**
```
docker build -t $REPOSITORY_URI:latest .
docker push $REPOSITORY_URI:latest
```

## Cloudformation
- **Stack Termination Protection** - can be enabled to prevent a stack from being accidentally deleted. Similarly, the **'cloudformation:DeleteStack' Action** applies to entire stack(s)
- The **DeletionPolicy** attribute can be used to preserve a specific resource when its stack is deleted
- The **DeletionPolicy Retain** option can be used to ensure AWS CloudFormation keeps the resource without deleting the resource
- cfn-init helper script can be used to install packages, create files, and start/stop services 

## Kinesis

## Cloudfront 

## ElastiCache
- CloudFront and ElastiCache are both caching technologies which can be used to improve performance of web applications, by caching data and content
- ElastiCache is the best option for storing session state as it is scalable, highly available and can be accessed by multiple web servers

## CloudWatch
- With detailed monitoring, data is available in 1-minute periods for an additional charge.
- If you want to collect metrics at 10 second intervals, you need to use high-resolution metrics
- CloudWatch dashboards are customizable home pages in the CloudWatch console that you can use to monitor your resources in a single view, even those resources that are spread across different Regions

## X-Ray
- In Amazon ECS, create a Docker image that runs the X-Ray daemon, upload it to a Docker image repository, and then deploy it to your Amazon ECS cluster
- X-Ray provides an official Docker container image that you can deploy alongside your application
- X-Ray can be used with applications running on EC2, ECS, Lambda, Amazon SQS, Amazon SNS and Elastic Beanstalk
-  If you’re using Elastic Beanstalk, you will need to include the language-specific X-Ray libraries in your application code. For applications running on other AWS services, such as EC2 or ECS, you will need to install the X-Ray agent and instrument your application code.


## Secrets Manager & Parameter Store
|Secrets Manager|Both|Parameter Store|
|--------------|----|----------------|
||Integrated with Identity and Access Management||
||Can store credentials in hierarchical form ||
||Supports encryption at rest using customer-owned KMS keys||
|enables you to store, retrieve, control access to, rotate, audit, and monitor secrets centrally||provides secure, hierarchical storage for configuration data management and secrets management|
|charges you for storing each secret|||
|provides a secret rotation service||||

