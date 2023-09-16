## 1. EC2
- Infra as service
- Storage volumes for temporary data that are deleted when you STOP or TERMINATE your instance, known as instance store volumes.
- Persistent storage volumes for your data using Elastic Block Store volumes.
- Instance states: Start, Stop, Hibernate, Terminate
- bootstrapping means launching commands when a machine starts
- User Data Script runs with the root users
- *Instance Store*: 
    - High-performance hardware disk, use EC2 Instance
    - Better I/O
    - Loss the data if ec2 stoped

## 2. ELB
- Types of load balancers:
    - *`Classic Load Balancer`* (v1 - old generation) – 2009 – CLB
        - HTTP, HTTPS, TCP, SSL (secure TCP) IPv4 & IPv6
        - Health checks are TCP or HTTP based
        - Fixed hostname: xxx.region.elb.amazonaws.com

    - *`Application Load Balancer`* (v2 - new generation) – 2016 – ALB
        - Layer 7 HTTP, HTTPS, WebSocket, HTTP/2
        - Support redirects (fixed hostname)
        - Routing tables to different target groups:
            • Routing based on path in URL (example.com/users and example.com/posts)
            • Routing based on hostname in URL (one.example.com and other.example.com)
            • Routing based on Query String, Headers (example.com/users?id=123&order=false)
        - Port mapping feature to redirect to a dynamic port in ECS
        - Application servers don’t see the IP of the client directly (it sees the IP of the load balancer instead)
        - `X-Forwarded-For` header contains the IP address of the client and `X-Forwarded-Port` contains the port from which the client connected to the load balancer.
        - **`ALB Access Logs`** capture detailed information about requests sent to your load balancer
    - *`Network Load Balancer`* (v2 - new generation) – 2017 – NLB
        - TCP, TLS (secure TCP), UDP
        - Extreme performance, static IP or Elastic IP
        - Less latency ~100ms (vs 400ms for ALB)

    - **Gateway Load Balancer** – 2020 – GWLB
        - Operates at layer 3 (Network layer) – IP Protoco
        - Used to deploy, scale and manage third-party virtual appliances such as firewalls, intrusion detection and prevention systems, analytics, etc.
        - Can be used for Network Address Translation (NAT)
        - Uses the `GENEVE` protocol on `port 6081`
        - Target group: EC2 (must be private IP)


- **Launch Configuration of elb:**
    

## 3. ASG - Auto Scaling Group
- Scale out (add EC2) to match an increased load
- Scale in (remove EC2) to match a decreased load
- Target tracking scaling: 
    - most simple and easy to set-up
    - i.e CPU usage, network in/out, request count
- Simple/Step scaling:
    - when a CloudWatch alarm is triggered (example CPU > 70%)
    - when a specific metric reaches a specific value
- Scheduled Actions:
    - Anticipate a scaling based on known usage patterns
    - i.e increase the min capacity to 10 at 5pm on Fridays
- ASG that spans across multiple regions need to be enabled for each region
- ASG that can contain EC2 instances in only one AZ of a region
- Unhealthy instances are terminated and replaced by new ones (ASG)

## 4. Elastic IP
- Public IPv4 IP you own as long as you don’t delete it
- You can attach it to one instance at a time
- Target groups: EC2 instances (must be public IP)

## 5. Security Group
- Security groups are acting as a “firewall” on EC2
- Only contain `allow` rules
- Security groups rules can reference by IP or by security group.
- If app isn't accessible (time out), then it’s a security group issue
- “connection refused“ error, then it’s an application error
- Inbound `blocked` and Outbound `Authorized` by default.
- Security groups are `stateful` (if you allow an inbound rule, automatically create an outbound rule).

## 6. EBS Volume
- (Elastic Block Store) Volume is a network drive you can attach
to your instances.
- Persistent data after termination
- Mounted on instance at a time 
- Locked to AZ, snapshot to move across AZs
- By default, the root EBS volume is deleted and others don't 
- Recycle Bin for EBS Snapshots
    • Setup rules to retain deleted snapshots so you can recover them after an accidental deletion
    • Specify retention (from 1 day to 1 year)
- 6 types:
    • gp2 / gp3 (SSD): General purpose SSD volume that balances price and performance for a wide variety of workloads
    • io1 / io2 (SSD): Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads
    • st1 (HDD): Low cost HDD volume designed for frequently accessed, throughput-intensive workloads
    • sc1 (HDD): Lowest cost HDD volume designed for less frequently accessed workloads
- Only gp2/gp3 and io1/io2 can be used as boot volumes.
- `1 GiB - 16 TiB` range of storage ( min - max )

## 7. EFS
- Managed Network file system (NFS)
- Works with EC2 instances in multi-AZ
- Uses: NFSv4.1 protocol
- 2 Performance modes: <br> 
    • General purpose (default) <br>
    • Max I/O - higher latency
- Throughput modes: <br>
    • Bursting (default) - scales as file system grows <br>
    • Provisioned - provision throughput regardless of storage size
    • Elastic - provision throughput as file system grows.

## 8. RDS
- Allows to create relational databases in the cloud
- List of supported engines: <br>
    • PostgreSQL <br>
    • MySQL <br>
    • MariaDB <br>
    • Oracle <br>
    • Microsoft SQL Server <br>
    • Aurora (AWS Proprietary database) <br>
### Advantages:
- Managed service: automated provisioning, OS patching, continuous backups, DB setup, configuration & patching
- Multi AZ setup for disaster recovery
- Can't ssh into the instances
- Have to set max storage threshold
- Auto modifies the storage if:
    • Free storage < 10% of allocated storage
    • Low storage lasts 5 minutes

### Read Replicas:
- Up to 5 read replicas
- Async replication so reads are eventually consistent
- Can be Multi AZ
- Must update connection string to leverage read replicas
- Only used for read only queries (no writes), except for Aurora MySQL 5.7

## 9. Aurora:
- Pg and MySQL compatible so drivers will work as if Aurora was a Postgres or MySQL db.
- Cloud Optimized (5x faster than MySQL, 3x faster than Postgres)
- Grows in increments of 10GB up to 128TB
- Have up to 15 replicas
- Cost 20% more than RDS.
- Storage stiped across 100s of volumes
- Failover in less than 30 seconds

## 10. Elastic Cache:
- In-memory data store
- Redis or Memcached
- Improve latency and throughput for read-heavy applications
- Cache often accessed data

## 11. Key Pairs
- Public and private key to connect to EC2 instances
- .pem file
- Don't lose the private key, you can't connect to your instance

## 12. ECS, ECR, EKS, Fargate
- ECS: Elastic Container Service used to run Docker containers on EC2 instances
- ECR: Elastic Container Registry used to store Docker images
- EKS: Elastic Kubernetes Service used to run Kubernetes on AWS
- Fargate: Serverless compute engine for containers (no EC2 instances to manage)
- ECS Task Placement Strategies:
    1. Binpack
    2. Random
    3. Spread
- ECS Task Placement Constraints:
    1. distinctInstance
    2. memberOf
- ECR is backed by S3
- EKS is way to launch managed Kubernetes clusters on AWS

## 13, Amazon MemoryDB:

- Over 160 million reads per second
- Scales seamlessly from 10s GB to 100s of TBs

## 14. Route53:

- DNS service
- The only service that is 100% available

    ### Route53 Record types:

    - A: maps to IPv4
    - AAAA: maps to IPv6
    - CNAME: maps to another DNS name
    - NS: name server for subdomains

- Except for alias records, ttl is mandatory

    ### Alias Records:

    - Maps a hostname to an AWS resource
    - Always type of A/AAAA ipv4/ipv6
    - Can't set ttl
    - Target can be ELB, CloudFront, S3, API Gateway, etc.

    ### Routing Policies:

    - List of routing policies:

        #### Simple (default)
        - Can specify multi values
        - Random one will be choosen
        - Can'r be assosiacted with health checks <br>

        #### Weighted
        - Control the % of the requests that go to each specific resource
        - Can be associated with health checks
        - traffic (%) = weight / total weight
        - Uses for A/B testing or canary deployments
        - Weight 0 = equivalent to a record not being returned

        #### Latency
        - Redirect to the server that has the least latency close to us
        - Latency is evaluated in terms of user to designated AWS region

        #### Failover
        - Health checks are mandatory
        - HTTP health check is for public
        - if > 18% reports healthy then its good.
        - Combine up to 256 child health checks
        - Private end point can't be used for health checks so have to setup cloud watch alarm and health check itself

        #### Geolocation
        - Based on user location

        #### Geoproximity
        - Route traffic to your resources based on the geographic location of your users and your resources
        - To shift traffic based to resources based on defined bias
        - `Trafic Flow` used to create advanced routing policies
        
        #### Multivalue
        - Used when routing traffic to multiple resources
        - Up to 8 healthy records are returned for each multi value query
        - Not a substitute for having an ELB

## 15. VPC:

1. IGW - Internet Gateway helps to connect to the internet
2. NAT Gateway - helps to connect to the internet from private subnet
3. NAT Instance - helps to connect to the internet from private subnet

## 16. S3
- Infinitely scalable
- Use cases:
    - Backup/Storage
    - Static website hosting
    - Disaster Recovery
    - Media hosting (videos, images, etc.)

- Max file size: 5TB
- if more than 5GB then use multi-part upload
- Enabling versioning will have version `null`
- Suspending versioning doesn't delete the previous versions
- CRR (Cross Region Replication) 
    - complience, lower latency, replication across accounts
- SRR (Same Region Replication)
    - log aggregation, live replication between production and test accounts
- To give access of object for particular time period use `Pre-Signed URL` 

- S3 Storage classes:
    - S3 Standard
    - S3 Intelligent Tiering
    - S3 Standard-Infrequent Access (IA)
    - S3 One Zone-Infrequent Access
    - S3 Glacier Instant Retreival
    - S3 Glacier Flexible Retreival
    - S3 Glacier Deep Archive

<br>

- S3 Lifecycle rules:
    - Transition action: move objects to another storage class after a certain period of time 
    - Expiration action: delete the object after a certain period of time
    - Can be applied to current versions and previous versions

- S3 Baseline Performance:
    - 3,500 PUT/COPY/POST/DELETE or 5,500 GET/HEAD requests per second per prefix in a bucket
    - If you spread reads across all the prefixes, you can achieve 22,000 read requests per second

- S3 Encryption:
    - With Amazon S3-Managed Keys (SSE-S3)
        - Enabled by default
        - Uses AES-256 encryption
        - Header: `x-amz-server-side-encryption: AES256`

    - With AWS KMS-Managed Keys (SSE-KMS)
        - user control + audit trail (who used the key to access which object)
        - Header: `x-amz-server-side-encryption: aws:kms`
        - Limitation of using KMS:
            1. Must be in the same region
            2. Additional charges from KMS
            3. `GenerateDataKey` and `Decrypt` API calls are needed to upload/download objects
    
    - With Customer-Provided Keys (SSE-C)
    - Client-Side Encryption
        - Uses client-side lib.
        - Must decrypt by themselves

## 17. CloudFront:

- DDos protection, integration with Shield, AWS Web Application Firewall

### CloudFront Origins:

- s3 bucket for distributing files and caching them at the edge
- Custom Origin (HTTP) for distributing dynamic content
- Invalidate cache using `CreateInvalidation` API.

### CloudFront vs S3 Cross Region Replication:

| CloudFront | S3 CRR |
| --- | --- |
| Global edge network | Must be setup for each region |
| Files are cached | Files are updated in near real-time |
| Great for static content | Great for dynamic content |

### CloudFront functions:

- Lightweight, written in `JS`, edge compute for modifying requests and responses
- Can be used to modify headers, redirect users, etc.

## 15. Elastic Beanstalk

- Managed service to deploy web applications

### Deployment modes:

1. Single instance deployment: good for dev
2. High availability with load balancing: prod

### Deployment policy:
    
#### All at once
1. Fastest deployment
2. Has downtime
3. No additional cost
#### Rolling
1. Update a batch of instances at a time
2. Can set the batch size and the pause time
3. App can run in both versions at the same time
#### Rolling with additional batch
1. Like rolling, but spins up new instances to move the batch
2. Good for prod
3. Both version runs simuntaneously
#### Immutable
1. Spins up new instances in a new ASG, deploys version, then swaps all instances when everything is healthy
2. Zero downtime
3. High cost, double capacity
4. Quick rollback
5. Great for prod

#### Blue/Green
1. Create a new "stage" environment and deploy version to it. Once healthy, swap URLs
2. Route53 weighted policy can be used to redirect a little bit of traffic to the stage environment

#### Traffic splitting
1. Refer as a `canary` deployment. Deploy a new version to a small portion of users, then monitor for errors. If everything is good, deploy to the rest of the users.
2. Can be done with CodeDeploy

#### Beanstalk Lifecycle Policy:
1. Can store at most 1000 app versions
2. Life cycle policy:
    - Based on time
    - Based on space

- Uses CloudFormation under the hood
- `RDS` can be provisioned with beanstalk, gr8 for dev/test but not for prod.
- Best way is to separately create RDS DB and provide connection string to `EB`

## 18. CloudFormation
> - Infra as a code
> - Can' t create dynamic amount of res.
### Templates Components:
1. Resources: your AWS resources declared in the template (MANDATORY)
    - Core of CF
    - Can reference each other
    - 224 resources types
2. Parameters: the dynamic inputs for your template
    - `!Ref` used to reference parameters
    - Pseudo parameters: `AWS::AccountId`, `AWS::Region`, `AWS::StackName`
3. Mappings: the static variables for your template
    - Used to map a key to a corresponding name
    - `!FindInMap` used to reference mappings
4. Outputs: References to what has been created
    - Used to reference resources created in the template
    - Can't delete a CF stack if there are outputs that are referenced by another stack
    - `!ImportValue`  used to reference outputs from another stack
5. Conditionals: List of conditions to perform resource creation
    - Used to control the creation of resources or outputs based on a condition
6. Metadata
    - Information about the template
- Valid parameters: `String`, `Number`, `List<Number>`, `CommaDelimitedList`, `AWS Parameter Types`

### CF Rollbacks:
- Stack creation fails:
    - Default: everything rolls back (gets deleted)
- Stack updation fails:
    - Automatically rolls back to the previous known working state

### CF Driffs:
- Used to track the manual changes

### Stack Policy:
- Prevent updates to certain resources
- JSON based
- Works only on update stack

> Note: An exported value in cloudformation must be unique within an AWS region ( Same region ) 

## 19. SQS
- Unlimited throughput and number of msg in queue
- Default retention of messages: `4 days, maximum of 14 days`
- Poll msgs up to 10 at a time.
- Delete the messages using the `DeleteMessage` API
- Scale consumer horizontally to improve
throughput of processing

### SQS Security:

- In-flight encryption using HTTPS API
- At-rest encryption using KMS keys
- Client-side encryption if the client wants
- SQS Access Policies (similar to S3 bucket policies)

### MSG Visibility Timeout:

- Default timeout: 30s
- If a message is not processed within the visibility timeout, it will be processed twice
- If visibility timeout is high (hours), and consumer crashes
- If visibility timeout is low, may get duplicates

### DLQ:

[PANDING...]

### SQS Long Polling:

- Long polling can be enabled at the queue level
- `ReceiveMessageWaitTimeSeconds` API used to enable long polling (1-20s)

### SQL Important APIs:

- `CreateQueue` (MessageRetentionPeriod), `DeleteQueue`
- `PurgeQueue`: delete all the messages in queue
- `SendMessage` (DelaySeconds), `ReceiveMessage`, `DeleteMessage`
- `MaxNumberOfMessages`: default 1, max 10 (for ReceiveMessage API)
- `ReceiveMessageWaitTimeSeconds`: Long Polling
- `ChangeMessageVisibility`: change the message timeout

## 20. SNS:

- Push notifications to mobile devices, email, SQS queues, HTTP endpoints
- Fan-out pattern: 1 message published to SNS topic and received by many SQS queues or HTTP endpoints
- Up to 12,500,000 subscriptions per topic
- 100,000 topics limit

### SNS vs SQS:

| SNS | SQS |
| --- | --- |
| Push mechanism | Pull mechanism |
| No need to poll | Need to poll |
| Send one message to many receivers | Each message goes to one receiver |
| Subscribers can be web servers, email addresses, SQS queues, Lambda functions, mobile devices | Subscribers can be SQS queues, Lambda functions, web servers |

## 21. Kinesis
### `Kinesis Data Streams`:
> Capture, store, and process data streams in real time
  
- Retention: 1 day to 365 days
- Can have multiple consumers
- Provisioned mode:
    - Shards: 1MB/s or 1000 msg/s at write PER SHARD (1MB/s OUT)
    - 2MB/s read PER SHARD across all consumers (2MB/s IN)
    - Need to choose number of shards
- On demand mode:
    - Default capacity is 4MBs in or 4000 records per sec

#### Kinesis Data Streams Security:

- HTTPS, KMS for encryption at rest
- VPC endpoints available for Kinesis Data Streams
- Monitor API calls using `CloudTrail`

### `Kinesis Data Firehose`: 
> Load data streams into S3, Redshift, ElasticSearch, Splunk
- Fully managed
- Near real time 60s latency or 1MB of data at a time
- Can send failed records to S3

### `Kinesis Data Analytics`: 
> Analyze data streams with SQL or Apache Flink
- Real time analytics using SQL
- Fully managed
- Use cases: real time dashboards, real time metrics, generate alerts, etc.

- `Kinesis Video Streams`: Capture, process, and store video streams.

## 22. Lambda

- Lambda tries 3 times in case of failures
- 1min wait after 1st attempt then 2min wait
- DLQ can be used to capture failed events
- Max execution time: 15 mins
- Max memory: 10GB
- Can't run more than 1000 concurrent executions
- Can't access the underlying infrastructure
- VPC endpoints can be used to access AWS services
- 1792MB of memory = 1 vCPU
- `/temp` directory is limited to 10GB and used for disk space if lambda needs to write something to disk
- `Lambda authorizer` can be used to authorize API Gateway requests
- Lambda can be deployed as a container up to 10GB from ECR
- **`Lambda Runtime Interface Emulator`** used to test lambda functions locally
- Must create the Lambda function from the same account as the container registry in Amazon ECR

### Lambda@Edge:

- Used to run Lambda functions at the edge locations of CloudFront to provide low latency to end users
- CloudFront provides 2 types:
1. Lambda@Edge
2. CloudFront Functions (newer)
- Mainly used to customize CDN behavior
- Written in Node.js and Python
- Scale 1000s of req/s

### CloudFront Functions vs Lambda@Edge:

|| CloudFront Functions | Lambda@Edge |
| --- | --- | --- |
| Runtime | JavaScript | Node.js, Python |
| Max execution time | < 1ms | 5-10s |
| Max memory | 2MB | 128MB up to 10GB |

### Lambda Layers:

- Used to share code and dependencies between functions
- Custom runtimes can be created using layers
- 5 layers per function up to 250MB

### Lambda Dependencies:

- Dependencies can be packaged with the function or in a Lambda Layer
- Upload a zip if < 50MB else in s3
- Can deploy lambda as container up to 10GB from ECR
- Can deploy lambda with code `deploy`, AppSpec file is executed by CodeDeploy to deploy the code.
- Env size limit: 4KB

## 23. DynamoDB

- max size of item: 400KB
- max size of partition key: 2048 bytes
- `“ProvisionedThroughputExceededException”` error occurs when limit is exceeded (read/write capacity)
- 1 WCU = 1 write per second for an item up to 1KB
- 1 RCU = 1 strongly consistent read per second, or 2 eventually consistent reads per second, for an item up to 4KB
> Example: 10 strongly consistent reads per second of 6kb items is demanded when how many RCU is needed?
> 10 * 6kb/4kb = 15 RCU
- When capacity is exceeded, requests are throttled
    - Solution: exponential backoff (retry with a delay)
    - Distribute par tition keys across multiple partitions

### DynamoDB PartiQL:

- SQL-compatible query language for Dynamo
- Can't do joins or aggregations (only SELECT, INSERT, UPDATE, DELETE)

### DynamoDB LSIs:

- Local Secondary Indexes
- Alt sort key for the table
- Up to 5 LSIs per table

### DynamoDB GSIs:

- Global Secondary Indexes
- Speeds up queries on non-key attributes
- Can be added / modified anytime
- If write capacity is exceeded, then write is throttled in main table and GSI

### DynamoDB Optimistic Locking:

- Prevents concurrent writes to the same item
- Uses a version number (attribute `“version”`) on each record
- If version number doesn’t match, then write is rejected

### DynamoDB Accelerator (DAX):

- In memory cache of Dynamo
- Solves `hot key` problem
- `5min` default TTL
- Up to 10 nodes in the cluster
- Multi AZ

### DynamoDB Streams:

- Stream of item lvl modification
- Retention: 24 hours by default, up to 7 days
- Expired item will be deleted within `48 hours`
- Can be used to trigger Lambda functions
- Streams can be enabled on the main table or GSI

### DynamoDB Transactions:

- Consumes 2x the WCU/RCU
- Up to 25 items or 4MB of data
- Two operations:
    > TransactGetItems: <br/>
    > one or more `GetItem` operations

    > TransactWriteItems: <br/>
    > one or more `PutItem`, `UpdateItem`, `DeleteItem` operations 
    > `TransactWriteItems` can contain up to 10 operations and used to update multiple items atomically

- Example1: 3 Transactional writes per second, with item size 5 KB
    > 3 ∗ (5kb/1kb) ∗ 2 (𝑡𝑟𝑎𝑛𝑠𝑎𝑐𝑡𝑖𝑜𝑛𝑎𝑙 𝑐𝑜𝑠𝑡) = 30 𝑊𝐶𝑈𝑠

## 24. API Gateway

- Used to create, publish, maintain, monitor, and secure APIs
- Can create APIs for:
    - REST APIs
    - WebSocket APIs
    - HTTP APIs (newer)
- Can be used to expose HTTP endpoints (including Lambda functions)
- Default caching TTL: 300s (min 0s, max 3600s)
- Can enable throttling to prevent attacks
- In case of throttling => `429 Too Many Requests`
- Cache capacity: `0.5GB - 237GB`
- Endpoint types:
    - Edge-optimized: default, for global clients
    - Regional: for clients within the same region
    - Private: for clients within the same VPC
- Can use API Gateway as a proxy for DynamoDB, S3, or HTTP endpoints

### HTTP API vs REST API:

| HTTP API | REST API |
| --- | --- |
| Cheaper | More features |
| Faster | More mature |
| Simpler | More integrations |
| Supports OIDC and JWT | Supports Cognito and Lambda Authorizers |

## 25. CICD

### CodeCommit:
- Source control service that hosts Git-based repositories
- Encryption at rest using KMS
- Encryption in transit using SSL
- IAM policies used to control access to repositories
- Triggers can be used to send notifications to SNS or invoke Lambda functions

### CodeBuild:
- Fully managed build service
- Used to compile source code, run tests, and produce software packages
- It can execute commands from `buildspec.yml` file, cmd can be `bash` or `powershell` scripts
- Can be used to build Docker images
- Charged per minute for compute resources
- Phases:
    - Install: install dependencies
    - Pre-build: final commands to execute before build
    - Build: build commands
    - Post-build: commands to execute after build
- Bundling the dependencies with the source code is a best practice to avoid long install times and to ensure that the build always uses the same dependencies

### CodeDeploy:

- File named `appspec.yml` used to manage deployments.
- Can be used to deploy to EC2, Lambda, ECS, on-premises instances
- Must run **CodeDeploy** agent on EC2 instances
- `“InvalidSignatureException` error occurs when time on EC2 and signature on CodeDeploy are not in sync.
- `validateService` API call used to check if CodeDeploy agent is running on EC2 instances

### CodeStar:

- All in one service for CICD

### CodeArtifact:

- Fully managed artifact repository
- Store and share software packages
- Supports Maven and npm packages

### CodeGuru:

- Supports Java and Python

## 26. SAM - Serverless Application Model

- `Transform: 'AWS::Serverless-2016-10-31'` indicates that this is a SAM template
- `Resources` section is where you define your Lambda functions, APIs, DynamoDB tables, etc.
- Intigrated with `CodeDeploy`
### SAM Commands:
- `sam build`: build the application
- `sam deploy`: deploy the application
- `sam package`: package the application
- `sam local invoke`: invoke the application locally
- `sam local start-api`: start the API locally
- `sam local start-lambda`: start the Lambda



## 27. IAM ( User, Group, Policy )
- User: are people within your organization, and can be grouped
- Groups only contain users, not other groups.
- In policy apply the least privilege principle.
- **`IAM Access Advisor`** shows the service permissions granted to a user and when those services were last accessed.
- **`IAM Access Analyzer`** used to analyze resource policies to help administrators and security teams protect their resources from unintended access.

### Policy Stucture:

- Version: policy language version, always include “2012-10-17”
- Id: an identifier for the policy (optional)
- Statement: one or more individual statements (required)
  - Statements consists of
    - Sid: an identifier for the statement (optional)
    - Effect: whether the statement allows or denies access (Allow, Deny)
    - Principal: account/user/role to which this policy applied to
    - Action: list of actions this policy allows or denies
    - Resource: list of resources to which the actions applied to
    - Condition: conditions for when this policy is in Effect

- AWS SDKs (JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++)

- IAM Credentials Report (account-level):
• a report that lists all your account's users and the status of their various credentials

- IAM Access Advisor (user-level)
• A:ccess advisor shows the service permissions granted to a user and when those services were last accessed.

## 28, CloudWatch
- Used to monitor AWS resources and applications
- In **EC2 detail monitoring** cloud watch will collect data every 1 min
- `cloudwatch high resolution` will collect data every 1 sec (for an additional cost)
- ``

### CloudWatch Custom Metrics:
- `PutMetricData` API call used to send custom metrics to CloudWatch
- Accepts metric data points two weeks in the past and two hours in the future

### CloudWatch Logs:
- **Log Groups**: arbitrary name, usually representing an application
- **Log Streams**: instances within application / log files / containers
- Expiration: never expire / 1 day to 10 years...
- Logs encrypted by default.
- Log data takes up to **`12 hours`** to become available for export
- `CreateExportTask` API call used to export logs to S3
- Need to run `CloudWatch Agent` on EC2 instances to send logs to CloudWatch (with IAM permissions)

### CloudWatch Unified Agent - Metrics:
- Collects system-level metrics from EC2 instances
- CPU, memory, disk, network

### CloudWatch Log insights:
- Used to query logs
- Query syntax: `fields @timestamp, @message | sort @timestamp desc | limit 20`
- Log insights can be used to create CloudWatch alarms

### CloudWatch Logs Metric Filters:
- Uses filter expressions to search for matches in log events

### CloudWatch Alarms:

- Used to trigger notifications for any metric
- Alarm states: OK, INSUFFICIENT_DATA, ALARM
- Alarm can trigger Auto Scaling, EC2 actions, SNS notifications
- Can be created from the console or using `PutMetricAlarm` API call and based on CloudWatch Logs Metrics Filters.
```bash
# demo command to set alarm state using aws cli
aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value
ALARM --state-reason "testing purposes"
```
- Composite alarms: multiple alarms combined together with AND/OR logic
- AWS requires approx `5 weeks` of usage to create a forecast


## 29. AWS Other Services

### AWS Cognito:

- Used to add authentication, authorization, and user management to web and mobile apps
- `User Pools`: used to manage sign-up and sign-in functionality for web and mobile apps
- `Identity Pools`: used to provide AWS credentials to users so they can access AWS resources directly
- `Federated Identities` used to enable users to sign in to your apps through a third party identity provider (Google, Facebook, etc.)
- `Cognito Sync` used to synchronize user data for a user across devices

### AWS Step Functions:
- Visual workflow service to orchestrate AWS Lambda functions
- Can be used to orchestrate other AWS services
- Visualize the workflow and track the execution as well as retry failed executions

#### Step Functions - States:
- Choice state: if/else logic
- Failed / Succeeded state: terminal states
- Pass state: pass input to output
- Wait state: wait before moving to the next state

#### Step Functions Standard vs Express Workflows:
|| Standard | Express |
| --- | --- | --- |
| Max execution time | 1 year | 5 mins |
| Execution Model | Event-driven | Request/response |
| Retry | Yes | No |

### AWS AppSync:
- Uses `GraphQL` to interact with data sources.
- Can be used to interact with DynamoDB, Lambda, Elasticsearch, HTTP endpoints, etc.
- Starts with uploaing a `schema.graphql` file

### AWS Amplify:
- It is a framework for building full-stack serverless apps
- `Amplify CLI` can be used to create and manage AWS resources
- `Amplify.yaml` is used to run tests, deploy, etc.

### AWS STS - Security Token Service:
- Allows to grant limited and temporary access to AWS resources

### AWS SES - Simple Email Service:

- Managed email service to send emails at scale

### AWS OpenSearch Service:

- Managed service to deploy, secure, and operate Elasticsearch clusters
- 2 modes: managed cluster and serverless cluster
- Doesn't natively support SQL
- Comes with dashboards and Kibana

### AWS Athena:

- Used to query data in S3 using SQL
- Supports **CSV, JSON, ORC, Avro, Parquet, etc**.

### AWS Managed Streaming for Apache Kafka (MSK):
- Serverless Apache Kafka
- Multi AZ cluster deployment

### AWS Certificate Manager (ACM):
- Easily provision, manage, and deploy SSL/TLS certificates
- Both public and private TLS
- Auto renew certificates

### AWS Macie:
- Managed data security and data privacy service
- Alerts when sensitive data is detected

### AWS Config:
- Independent from application code.
- Features: flags, app tuning, allow/deny, etc.

## 30, AWS Security Services:

### AWS KMS - Key Management Service:
- managed by AWS
- Can be used to encrypt data in S3, EBS, RDS, etc.
- Has a limit of 4KB for encryption
- More than 4KB, use `envelope encryption` which is also referred as `GenerateDataKey` API call
- `DEK` - Data Encryption Key 
- `CMK` - Customer Master Key
- KMS stores CMKs in a `key store` and encrypts the data received from the client using the CMK

### SSM Parameter Store:

- Secure storage for configuration and secrets
- Can be used to store passwords, database strings, API keys, etc.
- Can be used to store encrypted data
- Has a limit of 8KB for encryption

### AWS Secrets Manager:

- Meant for storing secrets
- `Rotation` feature to automatically rotate secrets
- Integrates with RDS.
- Secrets are encrypted using `KMS`

### AWS Nitro Enclaves:

- Isolated compute environment to process sensitive data.
- Uses `KMS` to encrypt data in transit