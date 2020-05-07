# AWS Lambda

* Serverless is a new paradigm in which the developers don't have to manage servers anymore
* Serverless does not mean there are no servers, it means you just don't manage/provision/see them.

### Serverless in AWS
* AWS Lambda & Step Functions
* DynamoDB
* AWS Cognito
* AWS API Gateway
* Amazon S3
* AWS SNS & SQS
* AWS Kinesis
* Aurora Serverless

## Lambda Overview

Benefits:

* Virtual **functions**
* Limited by time - **short executions**
* Run **on-demand**
* **Scaling** is automated
* Easy pricing:
	* Pay per request and compute time
	* Free tier of 1million AWS Lambda requests, 400k GBs compute time
* Integrated with the whole AWS stack
* Integrated with many programming language
* Easy monitoring through AWS CloudWatch
* Easy to get more resources per functions(up to 3GB of RAM)
* Increasing RAM will also improve CPU and network

### Configuration
* Timeout: default 3 seconds, max of 900s
* Environment Variables
* Allocated memory(128m to 3G)
* Ability to deploy within a VPC + asign security groups
* IAM Execution role must be attached to the Lambda function

## Concurrency and Throttling
* Concurrency: up to 1000 executions(can be increased through ticket)
* Can set a "reserved concurrency" at the function level
* Each invocation over the concurrency limit will trigger a "Throttle"
* Throttle Behaviour
	* if synchronous = return ThrottleError 429
	* if asynchronous = retry automatically and then go to DLQ

### Retries and DLQ
* If a lambda function asynchronous invocation fails, it will be retried twice
* After all retries, unprocessed events go to the Dead Letter Queue
* DLQ can be an SNS topic or SQS queue
* The original event payload is sent to the DLQ
* This is an easy way to debug what's wrong with your functions in production without changing the code
* Make sure the IAM execution role is correct for your Lambda function

## Logging, Monitoring and Tracing
* Cloudwatch
	* Lambda execution logs are stored in AWS CloudWatch Logs
	* Lambda metrics are displayed in AWS CloudWatch Metrics
	* Make sure your AWS Lambda function has an execution role with an IAM policy that authorizes writes to CloudWatch
* X-Ray
	* It's possible to trace Lambda with X-Ray
	* Enable in Lambda configuration(runs the X-Ray daemon for you)
	* Use AWS SDK in Code
	* Ensure Lambda functions has correct execution role

## Lambda Limits
* Execution
	* Memory allocation: 128 MB - 3008 MB (64MB increments)
	* Maximum execution time: 300 seconds (now 900 seconds)
	* Disk capacity in the "function container" (in /tmp): 512 MB
	* Concurrency Limit: 1000
* Deployment
	* Lambda function deployment size(compressed zip): **50 MB**
	* Size of uncompressed deployment(code + dependencies): **250 MB**
	* Can use the /tmp directory to load other files at startup
	* size of environment variables: 4 KB

## Versions	and Aliases
### Versions

* When you work on a Lambda function, we work on **$LATEST**
* When we're ready to publish a Lambda function, we create a version
* Versions are immutable
* Versions have increasing version numbers
* Versions get their own ARN
* Version = code + configuration(nothing can be changed - immutable)
* Each version of the lambda function can be accessed

### Aliases
* Aliases are "pointers" to Lambda function versions
* We can define a "dev", "test", "prod" aliases and have them point at different lambda versions
* Aliases are mutable
* Aliases enable Blue/Green deployment by assigning weights to lambda functions
* Aliases enable stable configuration of our event triggers/destinations
* Aliases have their own ARNs

## Lambda Function Dependencies
* **You need to install the packages alongside your code and zip it together**
* Upload the **zip** straight to Lambda if less than 50MB, else to S3 first
* Native libraries work: they need to be compiled on Amazon Linux

## Lambda and CloudFormation
* You must store the Lambda zip in S3
* You must refer the S3 zip location in the CloudFormation code

## Lambda Functions /tmp space
* If your lambda functions needs:
	* to download a big file to work
	* disk space to perform operations
	* max size 512MB
* The directory content remains when the execution context is frozen, providing transient cache that can be used for multiple invocations(helpful to checkpoint your work)
* For permanent persistence of object(non temporary), use S3

## Best Practices
* Perform heavy-duty work outside of your function handler
	* connect to databases outside of your function handler
	* Initialize the AWS SDK outside your function handler
	* Pull in dependencies or datasets outside of your function handler
* Use environment variables for
	* DB connection strings, S3 bucket, etc.. do not put these value directly in your code
	* Password, sensitive values can be encrypted using KMS
* Minimize your deployment package size to its runtime necessities
	* break down functions if need be
	* Remember the AWS Lambda limits
* Avoid using recursive code, never have a Lambda function call itself
* Don't put your function in a VPC unless you have to

## Lambda@Edge
* Deploying Lambda functions onto a CloudFront CDN globally
* Or how to implement request filtering before reaching your application

* Deploy Lambda functions alongside your CloudFront CDN
	* build more responsive applications
	* You don't manage servers, Lambda is deployed globally
	* Customize the CDN content
	* Pay only for what you use

* You can use Lambda to change CloudFront requests and responses:
* You can also generate responses to viewers without ever sending the request to the origin