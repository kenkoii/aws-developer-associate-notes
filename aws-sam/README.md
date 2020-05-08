# AWS Serverless Application Model(SAM)
Taking your Serverless Development to the next level

* Framework for developing and deploying serverless application
* All the configuration is YAML Code
* Generate complex CloudFormation from simple SAM YAML file
* Supports anything from CloudFormation: Outputs, Mapping, Parameters, Resources
* Only two commands to deploy to AWS
* SAM can use CodeDeploy to deploy Lambda functions
* SAM can help you to run Lambda, API Gateway, DynamoDB locally

## Recipe
* Transform header indicates it's SAM template:
	* `Transform: 'AWS::Serverless-2016-10-31'`
* WriteCode
	* AWS::Serverless::Function 
	* AWS::Serverless::Api
	* AWS::Serverless::SimpleTable
* Package & Deploy
	* `aws cloudformation package`/`sam package`
	* `aws cloudformation deploy`/`sam deploy`