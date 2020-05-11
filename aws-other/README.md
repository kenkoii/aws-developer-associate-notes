# Other AWS Services

## AWS CloudFront
* Content Delivery Network(CDN)
* Improves read performance, content is cached at the edge
* 136 Point of Presence globally(edge locations)
* Popular with S3 but works with EC2, Load Balancing
* Can help protect against network attacks
* SSL encryption(HTTPS) at the edge using ACM
* CloudFront can use SSL encryption(HTTPS) to talk to your applicaitons
* Support RTMP Protocol(videos/media)


## Step Functions
* Build serverless visual workflow to orchestrate your Lambda functions
* Represent flow as a JSON state machine
* Features: sequence, parallel, conditions, timeouts, error handling
* Can also integrate with EC2, ECS, On Premise servers, API Gateway
* Maximum execution time of 1 year
* Possibility to implement human approval feature
* Use cases:
	* Order fulfillment
	* Data processing
	* Web applications
	* Any workflow

## AWS SES
* Simple Email Service
* Send emails to people using:
	* SMTP
	* AWS SDK
* Ability to receiv email. Integrates with:
	* S3
	* SS
	* Lambda
* Integrated with IAM for allowing to send emails

## Databases
* RDS: Relational databases, OLTP
	* PostgreSQL, MySQL, Oracle...
	* Aurora + Aurora Serverless
	* Provisioned database
* DynamoDB: NoSQL DB
	* Managed, Key Value, Document
	* Serverless
* Elasticache: In Memory DB
	* Redis/Memcached
	* Cache Capability
* Redshift: OLAP - Analytic Processing
	* Data Warehousing/Data Lake
	* Analytics queries
* Neptune: Graph Database
* DMS: Database Migration Service

## AWS Certificate Manager(ACM)
* To host public SSL certificates in AWS, you can:
	* Buy your own upload them using the CLI
	* Have ACM provision and renew public SSL certificates for you (free of cost)
* ACM loads SSL certificates on the following integrations
	* Load Balancers(including one created by EB)
	* CloudFront distribution
	* APIs on API Gateway
* SSL certificates is overall a pain to manually manage, to ACM is great to leverage in your AWS infracture