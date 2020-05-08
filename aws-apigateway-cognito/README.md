# API Gateway and Cognito

## API Gateway
Build, Deploy and Manage APIs

* AWS Lambda + API Gateway = No infrastructure to manage
* Handle API Versioning(v1,v2,...)
* Handle different environments(dev, test, prod)
* Handle Security(authentication and authorization)
* Create API keys, handle request throttling
* Swagger/Open API import to quickly define APIs
* Transform and validate requests and responses
* Generate SDK and API specifications
* Cache API Responses

### Integrations
* Outside VPC:
	* AWS Lambda(most popular/powerful)
	* Endpoints on EC2
	* Load Balancers
	* Any AWS Service
	* External and publicly accessible HTTP endpoints
* Inside VPC:
	* AWS Lambda in your VPC
	* EC2 endpoints in your VPC

### Deployment Stages
* Making changes in the API Gateway does not reflect right away
* You need to make a `deployment` for them to be in effect
* Changes are deployed to **Stages**(as many as you want)
* Common naming are (dev, test, prod)
* Each stage has its own configuration parameters
* Stages can be rolled back as a history of deployment is kept

### Stage Variables
* Stage variables are like `environment variables` for API Gateway
* Used for often changing config variables
* They can be used in:
	* Lambda Function ARN
	* HTTP Endpoint
	* Parameter mapping templates
* Use cases:
	* Configure HTTP endpoints your stages talk to
	* Pass config parameters to AWS Lambda through mapping templates
* Stage variables are passed to the "context" object in AWS Lambda

### Stage Variables and Lambda Aliases
* We create a stage variable to indicate the corresponding Lambda alias
* Our API gateway will automatically invoke the right Lambda function 

### Canary Deployment
* Possibility to enable canary deployments for any stage(usually prod)
* Choose the % of traffic the canary channel receives
* Metrics and Logs are separate(for better monitoring)
* Possibility to override stage variables for canary
* This is blue/green deployment with AWS Lambda and API Gateway

### API Gateway Mapping Templates
* Mapping templates can be used to modify request/responses
* Rename parameters
* Modify body content
* Add headers
* Map JSON to XML for sending to backend or back to client
* Uses Velocity Template Language(VTL): for loop, if, etc..
* Filter output results(remove unnecessary data)

### Swagger/Open API
* Common way of defining REST APIs, using API definition as code
* Import existing Swagger/OpenAPI 3.0 spec to API Gateway
	* Method
	* Method Request
	* Integration Request
	* Method Response
	* + AWS extensions for API gateway and setup every single option
* Can export current API as Swagger/OpenAPI spec
* Swagger can be written in YAML or JSON
* Using Swagger we can generate SDK for our applications

### API Gateway Caching
* Caching reduces the number of calls made to the backend
* Similar to Elasticache and DynamoDB DAX
* Default TTL is 300 secs(min 0s, max:3600s)
* Caches are defined per stage
* Cache encryption option
* Cache capacity between 0.5GB to 237GB
* Possible to override cache settings for specific methods
* Able to flush entire cache immediately(invalidate it)
* Clients can invalidate cache with header: `Cache-Control:max-age=0` (requires proper IAM authorization)

### Logging, Monitoring, Tracing
* **CloudWatch Logs** 
	* Enable CloudWatch logging at the stage level(with log level)
	* Can override settings on a per API basis(ex. ERROR, DEBUG, INFO)
	* Log contains information about request/response body
* **CloudWatch Metrics**
	* Metrics are by stage
	* Possibility to enable detailed metrics
* **X-Ray**
	* Enable tracing to get extra information about requests in API Gateway
	* X-Ray API Gateway + AWS Lambda gives you the full picture

### CORS	
* CORS must be enabled when you receive API calls from another domain
* The OPTIONS pre-flight request must contain the following headers:
	* `Access-Control-Allow-Methods`
	* `Access-Control-Allow-Headers`
	* `Access-Control-Allow-Origin`
* CORS can be enabled through the console

### Usage plans & API Keys
* We can limit customer usage of our API
* **Usage plans**:
	* Throttling: set overall capacity and burst capacity
	* Quotas: number of requests made per day / week / month
	* Associate with desired API Stages
* **API Keys**:
	* Generate one per customer
	* Associate with usage plans
* Ability to track usage for API Keys

### Security
**IAM Permissions**

* Create an IAM policy authorization and attach to User/Role
* API Gateway verifies IAM permissions passed by the calling application
* Good to provide access within your own infrastructure
* Leverages `Sig v4` capability where IAM credential are in headers

**Lambda Authorizer(formerly Custom Authorizers)**

* Uses AWS Lambda to validate token in header being passed
* Option to cache result of authentication
* Helps to use OAuth/SAML/3rd party type of authentication
* Lambda must return an IAM policy for the user

**Cognito User Pools**

* Cognito fully manages user lifecycle
* API Gateway verifies identity automatically from AWS Cognito
* No custom implementation required
* Cognito only helps with **authentication**, not **authorization**

### Security Summary
* **IAM**:
	* Great for users/roles already within your AWS account
	* Handle authentication + authorization
	* Leverages **Sig v4**
* **Custom Authorizer**:
	* Great for 3rd party tokens
	* Very flexible in terms of what IAM policy is returned
	* Handle Authentication + Authorization
	* Pay per Lambda invocation
* **Cognito User Pool**:
	* You manage your own user pool(can be backed by Facebook, Google login, etc...)
	* No need to write any custom code
	* Must implement authorization in the backend

## AWS Cognito
Cognito is used when we want to give our users an identity so that they can interact with our application

* **Cognito User Pools**
	* Sign in functionality for app users
	* Integrate with API Gateway for authentication
	* Creates a serverless database of user for your mobile apps
	* Simple login(usename or email/password combination)
	* Possibility to verify emails/phone numbers and add MFA
	* Can enable Federated Identities(Facebook, Google, SAML)
	* Sends back a JSON Web Token(JWT)
* **Cognito Identity Pools(Federated Identity)**
	* Provide AWS credentials to users so they can access AWS resources directly
	* Log in to federated identity provider or remain anonymous
	* Get temporary AWS credentials back from the Federated Identity Pool
	* These credentials come with a pre-defined IAM policy stating their permissions
	* Ex: provide temporary access to write to S3 bucket using Facebook login
	* Integrate with Cognito User Pools as an identity provider
* **Cognito Sync**
	* Synchronize data from device to Cognito
	* May be deprecated and replaced by AppSync
	* Store preferences, configuration, state of app
	* Cross device synchronization(any platform - iOS, android, etc)
	* Offline capability(synchronization when back online)
	* Requires Federated Identity Pool in Cognito(not User Pool)
	* Store in datasets(up to 1MB)
	* Up to 20 datasets