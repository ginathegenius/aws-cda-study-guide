# AWS Certified Developer Associate
## Table of Contents
- [Serverless](#serverless)
  - [DynamoDB](#dynamodb)
  - [SQS](#sqs)
  - [S3](#s3)
- [Elastic Beanstalk](#elastic-beanstalk)
- [IAM](#iam)
- [Cognito](#cognito)
- [Security Token Service](#security-token-service)
- [ECS](#dynamodb)
- [CI/CD](#ci-cd)

## Serverless
- Lambda, Fargate, EventBridge, Step Functions, SQS, SNS, API Gateway, AppSync, S3, DynamoDB, RDS Proxy, Aurora Serverless
### DynamoDB
- Primary key attribute must be defined as type string, number, or binary

#### Query vs Scan 
| Query | Scan|
|---|--|
|finds items based on primary key values|reads every item in a table or a secondary index|

#### Indexes
| Local Secondary Index | Global Secondary Index|
|-----------------------|-----------------------|
| Must be created on table creation|Can be created at any time|
|Same partition key as table|Different partition key|
|Different sort key|Different sort key|

#### Provisioned Throughput 
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

When a consumer receives and processes a message from the queue, the message remains in the queue. SQS does not automatically delete the message. When the message is received, it remains in the queue. To prevent other consumers from processing the message again, Amazon SQS set a visibility timeout, a period of time that prevents other consumers from receiving and processing the message. The default is 30 seconds, min is 0 sec and max is 12 hr.

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
- AWS recommends that you use Multipart Upload for files larger than 100MB
- S3 Transfer Acceleration is recommended to increase upload speeds and especially useful in cases where your bucket resides in a Region other than the one in which the file transfer was originated
#### Encryption
- If file is to be encrypted at upload time, the x-amz-server-side-encryption parameter will be included in request header 
- Server-Side Encryption
    - S3 Managed Keys – SSE-S3 
    - AWS Key management Services KMS, managed keys - SSE-KMS 
    - Server side encryption with customer provided keys – SSE-C 
- Client Side Encryption
    - Customer encrypts files before upload by leveraging AWS Encryption SDK

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
    <li>All instances are out of service while deployment takes place </li>
    </ul></td>
  </tr>
  <tr>
    <th >Rolling</th>
    <td><ul><li> Deploys new version in batches </li>
    <li>Deploys new version in batches </li>
    <li>Reduced capacity during deployment  </li>
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

#### Cognito Events

### Identity pools
Identity pools enable you to grant your users access to other AWS services.



## Security Token Service
### AssumeRoleWithWebIdentity
STS AssumeRoleWithWebIdentity returns a set of temporary security credentials for users who have been authenticated in a mobile or web application with a web identity provider.

## ECS 

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
- Termination Protection - can be enabled to prevent a stack from being accidentally deleted
- cfn-init helper script can be used to install packages, create files, and start/stop services 

## Kinesis
