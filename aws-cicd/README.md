# AWS CICD
## Tech Stack for CICD
* Code 
	* AWS CodeCommit
	* Github or 3rd party code repository(bitbucket, gitlab)
* Build and Test
	* AWS CodeBuild
	* Jenkins CI or 3rd party(circle ci, travis, github actions)
* Deploy and Provision
	* AWS Elastic Beanstalk
	* AWS CodeDeploy
	* CloudFormation
* Orchestration
	* AWS CodePipeline
## CodeCommit
* Private Git repositories
* No size limit on repositories(scale seamlessly)
* Full managed, highly available
* Code only in AWS Cloud account => increased security and compliance
* Secure(encrypted, access control, etc...)
* Integrated with Jenkins/CodeBuild/other CI tools

### CodeCommit Security
* Interactions are done using Git(standard)
* Authentication in Git:
	* SSH keys: AWS Users can configure SSH keys in their IAM Console
	* HTTPS: Done through the AWS CLI Authentication helper or Generating HTTPS credentials
	* MFA
* Authorization in Git:
	* IAM Policies manage user/ roles right to repositories
* Encryption
	* Repositories are automatically encrypted at rest using KMS
	* Encrypted in transit(can only use HTTPS or SSH - both secure)
* Cross Account access:
	* Do not share SSH keys
	* Do not share AWS Credentials
	* Use IAM role in your AWS Account and use AWS STS(with AssumeRole API)

### CodeCommit Notifications
* You can trigger notifications in CodeCommit using **AWS SNS** (Simple Notification Service) or **AWS Lambda** or **AWS CloudWatch Event Rules**
* SNS/Lambda usecase:
	* Deletion of branches
	* Trigger for pushes that happen in master branch
	* Notify external Build System
	* Trigger AWS Lambda function to perform codebase analysis(maybe credentials got committed in the code)
* Cloudwatch Event Rules usecase:
	* Trigger for pull request updates
	* Code review
	* Commit comment events
	* CloudWatch Event Rules goes into an SNS topic

## CodePipeline
* Continuous Delivery
* Visual Workflow
* Source: Github/CodeCommit/Amazon S3
* Build: CodeBuild/Jenkins/etc...
* Load Testing: 3rd Party tools
* Deploy: AWS CodeDeploy/Beanstalk/CloudFormation/ECS
* Made of Stages:
	* Each stage can have sequential actions and / or  parallel actions
	* Stages can have 1 or more action groups
	* Each action groups can have 1 or more actions (for parallel actions)
	* Stages examples: Build/Test/Deploy/LoadTest/etc...
	* Manual approval can be defined at any stage

### CodePipeline Artifacts
* Each pipeline stage can create `artifacts`
* Artifacts are passed stored in Amazon S3 and passed on to the next stage

### CodePipeline Troubleshooting
* CodePipeline state changes happen in **AWS CloudWatch Events**, which can in return create SNS notifications
	* Ex. you can create events for failed pipelines
	* Ex. you can create events for cancelled stages
* If CodePipeline fails at a stage, your pipeline stops and you can get information in the console
* AWS CloudTrail can be used to audit AWS API calls
* If Pipeline can't perform an action, make sure the `IAM Service Role` attached does have enough permissions(IAM Policy)

## CodeBuild
* Fully managed build service
* Alternative to other build tools such as Jenkins
* Continuous scaling(no servers to manage or provision - no build queue)
* Pay for usage: the time it takes to complete the builds
* Leverages Docker under the hood for reproducible builds
	* AWS Managed or custom
* Possible to extend capabilities leveraging our own base Docker images
* Secure: Integration with KMS for encryption of build artifacts, IAM for build permissions, and VPC for network security, CloudTrail for API calls logging
* Source Code from Github/CodeCommit/CodePipeline/S3
* Build instructions can be defined in code(`buildspec.yml` file)
* Output logs to Amazon S3 & AWS CloudWatch Logs
* Metrics to monitor CodeBuild statistics
* Use CloudWatch Alarms to detect failed builds and trigger notifications
* SNS notifications
* Ability to reproduce CodeBuild locally to troubleshoot in case of errors
* Build can be defined within CodePipeline or CodeBuild itself

### CodeBuild Support environments
* Java
* Ruby
* Python
* Go
* Node.js
* Android
* .NET Core
* PHP
* Docker

### CodeBuild BuildSpec
* `buildspec.yml` file must be at the **root** of your code
* Define environment variables:
	* Plaintext variables
	* Secure secrets: use SSM Parameter store
* Phases(specify commands to run):
	* Install: install dependencies
	* Pre build: final commands to execute before build
	* Build: actual build commands
	* Post build: finishing touches(zip output for example)
* Artifacts: What to upload to S3(encrypted with KMS)
* Cache: Files to cache(usually dependencies) to S3 for future build speedup

### CodeBuild Local Build
* Running CodeBuild locally on you machine
	* Requires docker
	* Leverage the CodeBuild Agent

## CodeDeploy
* Deploy application automatically to many EC2 Instances
* These instances are not managed by Elastic Beanstalk
* Several ways to handle deployments:
	* Ansible
	* Terraform
	* Chef
	* Puppet
* Managed service

### Making CodeDeploy work
* Each EC2 machine(or On Premise) must be running the CodeDeploy Agent
* The agent is continuously polling AWS CodeDeploy for work to do
* CodeDeploy sends `appspec.yml` file
* Applicaiton is pulled from Github or S3
* EC2 will run the deployment instructions
* CodeDeploy Agent will report of success/failure of deployment on the instance

### Other
* EC2 instances are grouped by deployment group(dev/test/prod)
* Lots of flexibility to define any kind of deployments
* CodeDeploy can be chained into CodePipeline and use artifacts from there
* CodeDeploy can re-use existing setup tools, works with any application,
* auto scaling integration
* Note: Blue/Green only works with EC2 instances(not on premise)
* Support for AWS Lambda deployments
* CodeDeploy does not provision resources

### CodeDeploy Primary Concepts
* **Application**: unique name
* **Compute platform**: EC2/On-Premise or Lambda
* **Deployment configuration**: deployment rules for success/failure
	* EC2/On-Premise: you can specify the minimum numner of health instances for the deployment
	* AWS Lambda: specify how traffic is routed to your updated Lamda function versions
* **Deployment group**: group of tagged instances
* **Deployment type**: In-place deployment or Blue/Green deployment
* **IAM instance profile**: need to give EC2 the permission to pull from S3/Github
* **Application Revision**: application code + appspec.yml file
* **Service role**: Role for CodeDeploy to perform what it needs
* **Target revision**: Target deployment application version

### CodeDeploy AppSpec
* File section: how to source and copy from S3/Github to filesystem
* Hooks: set of instructions to do to deploy the new version(hook can have timeouts):
	* ApplicationStop
	* DownloadBundle
	* BeforeInstall
	* AfterInstall
	* ApplicationStart
	* ValidateService: really important(kind of like Health Check)
* Configs:
	* One at a time
	* half at a time
	* All at once
	* Custom
* Failures:
	* Instances stay in "failed state"
	* New deployments will first be deployed to "failed state" instances
	* To rollback: redeploy old deployment or enable automated rollback for failures
* Deployment Targets
	* Set of EC2 instances with tags
	* Directly to an ASG
	* Mix of ASG/Tags so you can build deployment segments
	* Customization in scripts with `DEPLOYMENT_GROUP_NAME` environment variables

## CodeStar
* CodeStar is an integrated solution that regroups: GitHub, CodeCommit, CodeBuild, CodeDeploy, CloudFormatiom, CodePipeline, Cloudwatch
* Helps create `CICD-ready` projects for EC2, Lambda, Beanstalk
* Issue Tracking integration with: Jira/Github issues
* Ability to integrate with Cloud9 to obtain a web IDE(not all regions)
* One dashboard to view all your components
* Free service, pay only for the underlying usage of the serviecs
* Limited Customization
