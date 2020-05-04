# AWS ElasticBeanStalk
* ElasticBeanStalk is a developer centric view of deploying an application on AWS
* It uses all the underlying component's we've seen before: EC2, ASG, ELB, RDS, etc...
* All in one view and still have full control over the configuration
* BeanStalk is free but you pay for the underlying instances
* Managed service
	* Instance configuration / OS is handled by beanstalk
	* Deployment strategy is configurable but performed by ElasticBeanStalk
* Just the application code is the responsibility of the developer
* Three architecture models:
	* **Single instance deployment**: good for dev
	* **LB + ASG**: great for production or pre-production web application
	* **ASG only**: great for non-web apps in production (workers, etc.)
* ElasticBeanStalk has three components
	* Application
	* Application version: each deployment gets assigned a version
	* Environment name (dev,test,prod..):free naming
* You deploy application versions to environments and can promote application versions to the next environment
* Rollback to feature to previous application version
* Full control over lifecycle of environments
* Support for many platforms:
	* Go
	* Java SE
	* Java with Tomcat
	* .NET on Windows Server with IIS
	* Node.js
	* PHP
	* Python
	* Ruby
	* Packer Builder
	* Docker

## Elastic Beanstalk Deployment Modes
* Single Instance
	* great for dev
* High Availability with Load Balancer
	* great for prod

### Deployment Options for Updates
* **All at once**(deploy all in one go) - fastest, but instances aren't available to serve traffic for a bit(downtime)
	* no additional cost
* **Rolling**: update a few instances at a time(bucket), and then move onto the next bucket once the first bucket is healthy
	* bucket = batch
	* zero downtime
	* will run under capacity 
	* additional cost
* **Rolling with additional batches**: like rolling, but spins up new instances to move the batch(so that the old application is still available)
	* good for prod
	* zero downtime
	* will always run in capacity(sometimes over)
	* additional cost
* **Immutable**: spins up new instances in a new ASG, deploy version to these instances, and then swaps all the instances when everything is healthy
	* zero downtime
	* high cost, double capacity
	* longest deployment
	* quick rollback (terminate new instances)
	* great for prod
* **Blue/Green**
	* Not a direct feature of EB
	* Zero downtime and release facility
	* Create new "stage" environment and deploy v2 there
	* The new environment(green) can be validated independently in our own time and roll back if issues
	* Route 53 can be setup using weighted policies to redirect a little bit of traffic to the stage environment
	* Using Beanstalk, "swap URLs" when done with the environment test

## Elastic Beanstalk Extensions
* A zip file containing our code must be deployed to Elastic Beanstalk
* All the parameters set in the UI can be configured with code using files
* Requirements:
	* in the `.ebextensions/` directory in the root of source code
	* `YAML/JSON` format
	* `.config` extensions (example: logging.config)
	* Able to modify some default settings using: option_settings
	* Ability to  add resources such as RDS, ElastiCache, DynamoDB, etc...
* Resources managed by `.ebextensions` get deleted if the environment goes away

## Elastic Beanstalk under the hood
* EBS relies on AWS CloudFormation

## Elastic Beanstalk Exam Tips

### HTTPS 
* Beanstalk with HTTPS
	* Idea: Load the SSL certificate onto the Load Balancer
	* Can be done from the Console(EB console, load balancer configuration)
	* Can be done from the code: `.ebextensions/securelistener-alb.config`
	* SSL Certificate can be provisioned using ACM(AWS Certificate Manager)
	* Must configure a security group rule to allow incoming port 443(HTTPS port)
* Beanstalk redirect HTTP to HTTPS
	* Configure your instances to redirect HTTP to HTTPS
	* or configure the ALB with a rule to redirect from HTTP to HTTPS
		* Make sure health checks are not redirected(so they keep giving 200 status code)

### Beanstalk Lifecycle Policy
* Elastic Beanstalk can store at most 1000 application versions
* If you don't remove old versions, you won't be able to deploy anymore
* To phase out old application versions, use a lifecycle policy
	* Based on time(old versions are removed)
	* Based on space(when you have too many versions)
* Versions that are currently used won't be deleted
* Option not to delete the source bundle in S3 to prevent data loss(**retention option**)

### Web Server vs Worker Environment
* If your application performs tasks that are long to complete, offload these tasks to a dedicated **worker environment**
* Decoupling your application into two tiers is common
* Example: processing a video, generating a zip file, etc
* You can define periodic tasks in a file `cron.yaml`

### RDS with Elastic Beanstalk
* RDS can be provisioned with Beanstalk, which is great for dev/test
	* This is not great for prod as the database lifecycle is tied to the Beanstalk environment lifecycle
	* The best for prod is to separately create an RDS database and provide our EB application with the connection string
* How to migrate from RDS coupled in EB to standalone
	* Create an RDS DB snapshot
	* Enable deletion protection in RDS
	* Create a new environment without an RDS, point to existing old RDS 
	* Perform blue/green deployment and swap new and old environments
	* Terminate the old environment(RDS won't get deleted because of deletion protection)
	* Delete CloudFormation Stack(will be in DELETE_FAILED state)
