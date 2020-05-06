# Monitoring
## CloudWatch
* Metrics: Collect and track key metrics
* Logs: Collect, monitor, analyze and store log files
* Events: Send notifications when certain event happen
* Alarms: React in real-time to metric/events

## Cloudwatch Metrics
* **Metric** is a variable to monitor(CPUUtilization, NetworkIn, etc..)
* Metrics belong to **namespaces**
* **Dimension** is an attribute of a metric(instance id, environment, etc...)
* Up to 10 dimensions per metric
* Metric have **timestamps**
* Can create CloudWatch dashboards of metrics 

### CloudWatch EC2 Detailed monitoring
* EC2 instance metrics have metrics "every 5 minutes"
* With detailed monitoring(for a cost), you get data "every 1 minute"
* Use detailed monitoring if you want to more prompt scale your ASG
* AWS Free tier allows us to have 10 detailed monitoring metrics
* EC2 Memory usage is by default not pushed(must be pushed from inside the instance as a custom metric)

### AWS CloudWatch Custom Metrics
* It's possible to define and send you own custom metrics to CloudWatch
* Ability to use dimensions to segment metrics
	* Instance.id
	* Environment.name
* Metric resolution:
	* Standard: 1 minute
	* High Resolution: up to 1 second(**StorageResolution** API parameter) - Higher cost
* Use API call PutMetricData
* Use exponential back off in case of throttle errors

## CloudWatch Alarms
* Alarms are used to trigger notifications for any metric
* Alarms can go to Auto Scaling, EC2 Actions, SNS notification
* Various options(sampling, %, max, min, etc..)
* Alarm States:
	* OK
	* INSUFFICIENT_DATA
	* ALARM
* Period:
	* length of time in seconds to evaluate the metric
	* High resolution custom metrics: can only choose 10 sec or 30 sec

## CloudWatch Logs
* Applications can send logs to CloudWatch using the SDK
* CloudWatch logs can go to
	* Batch exported to S3 for archival
	* Stream to ElasticSearch cluster for further analysis
	* Stream to lambda
* CloudWatch Logs can use filter expressions
* Logs storage architecture:
	* Log groups: arbitrary name, usually representing an application
	* Log stream: instances within application/log files/containers
* Can define log expiration policies(never expire, 30 days, etc..)
* Using the AWS CLI we can tail CloudWatch logs
* To send logs to CloudWatch, make sure IAM permissions are correctly set
* Security: encryption of logs using KMS at the Group Level

## CloudWatch Events
* Schedule: Cron jobs
* Event Pattern: Event rules to react to a service doing something
	* Ex. CodePipeline state changes
* Triggers to Lambda functions, SQS/SNS/Kinesis Message
* CloudWatch Event creates a small JSON Document to give information about the change

## AWS X-Ray
* Visual analysis of our applications
* Troubleshoot performances(bottlenecks)
* Understand dependencies in a microservice architecture
* Pinpoint service issues
* Review request behavior
* Find errors and exceptions
* Leverages **tracing**
	* Tracing is an end to end way to following a "request"
	* each component dealing with the request adds its own "trace"
	* Tracing is made of segments(+ subsegments)
	* Annotations can be added to traces to provide extra-information
* Ability to trace:
	* every request
	* sample request(as a % for example or rate per minute)
* X-Ray Security
	* IAM for authorization
	* KMS for encryption at rest

### X-Ray compatibility
* Lambda, Beanstalk, ECS, ELB, API Gateway, EC2 Instances, etc...

### Enabling X-Ray
1. Your code (Java, Python, Go, NodeJS, .NET) must import the AWS X-Ray SDK
	* very little code modification needed
	* The application SDK will then capture:
		*  calls to AWS Services
		*  HTTP/HTTPS request
		*  DB calls (MySQL, PostgreSQL, DynamoDB)
		*  Queue calls(SQS)
2. Install the X-ray daemon or enable X-Ray AWS Integration
	* X-Ray daemon works as a low level UDP packet interceptor
	* AWS lambda/other AWS services already run the X-Ray Daemon for you
	* Each application must have the IAM Rights to write data to X-Ray

* X-Ray service collects data from all the different services
* Service map is computed from all the segments and traces

### X-Ray Troubleshooting
* If X-Ray is not working on EC2
	* Ensure the EC2 IAM role has the proper permissions
	* Ensure the EC2 instance is runnign the X-Ray Daemon
* To enable on AWS Lambda
	* Ensure it has an IAM Execution role with proper policy(**AWSXRayWriteOnlyAccess**)
	* Ensure that X-Ray is imported in the code

### X-Ray Additional Exam Tips
* The X-Ray daemon/agent has a config to send traces cross account:
	* make sure the IAM permissions are correct - the agent will assume the role
	* This allows to have a central account for all your application tracing
* **Segments**: each application / service will send them
* **Trace**: segments collected together to form an end-to-end trace
* **Sampling**: decrease amount of request sent to X-Ray, reduce cost
* **Annotations**: Key Value pairs used to **index** traces and use with filters
* **Metadata**: Key Value pairs, _not_ indexed, not used for searching
* Code must be instrumented to use the AWS X-Ray SDK(interceptors, handlers, http clients)
* IAM role must be correct to send traces to X-Ray
* **X-Ray on EC2/On-premise**
	* Linux system must run the X-Ray daemon
	* IAM instance role if EC2, other AWS credentials on on-premise instance
* **X-Ray on Lambda**
	* Make sure X-Ray integration is ticked on Lambda(Lambda runs the daemon)
	* IAM Role is a Lambda role
* **X-Ray on Beanstalk**
	* set configuration on EB console
	* Or use a beanstalk extension(.ebextension/xray-daemon.config)
* **X-Ray on ECS/EKS/Fargate(Docker)**
	* Create a docker image that runs the daemon/or use the official X-Ray Docker image
	* Ensure port mappings & network settings are correct and IAM task roles are defined

## CloudTrail
* Track User activity and API usage
* Provides governance, compliance and audit for your AWS Account
* CloudTrail is enabled by default
* Get a history of events/API calls made within your AWS Account by:
	* Console
	* SDK
	* CLI
	* AWS Services
* Can put logs from CloudTrail into CloudWatch logs
* If a resource is deleted in AWS, look into CloudTrail first

## Summary
* CloudTrail:
	* Audit API calls made by users/servies/AWS Console
	* Useful to detect unauthorized calls or root cause of changes
* CloudWatch:
	* CloudWatch Metrics over time for monitoring
	* CloudWatch Logs for storing application log
	* CloudWatch Alarms to send notifications in case of unexpected metrics
* X-Ray
	* Automated Trace Analysis & Central Service Map Visualization
	* Latency, errors, fault analysis
	* Request tracking across distributed systems
