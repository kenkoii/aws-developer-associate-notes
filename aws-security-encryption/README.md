# AWS Security and Encryption
KMS, Encryption SDK, SSM Parameter Store

## Encryption 101
**Why encryption?**
### Encryption in flight (SSL)
* Data is encrypted before sending and decrypted after receiving
* SSL certificates help with encryption(HTTPS)
* Encryption in flight ensures no MITM(man in the middle attack) can happen

### Server side encryption at rest
* Data is encrypted after being received by the server
* Data is decrypted before being sent
* It is stored in an encrypted form thanks to a key(usually a data key)
* The encryption/decryption must be managed somewhere and the server must have access to it

### Client side encryption
* Data is encrypted by the client and never decrypted by the server
* Data will be decrypted by a receiving client
* The server should not be able to decrypt the data
* Could leverage `Envelope Encryption`

## KMS
AWS Key Management Service

* Anytime you hear "encryption" for an AWS service, it's most likely KMS
* Easy way to control access your data, AWS manages keys for us
* Fully integrated with IAM for authorization
* Can use CLI/SDK to encrypt

* Anytime you need to share sensitive information... use KMS
	* Database password
	* Credentials to an external service
	* Private Key of SSL certificates
* The value in KMS is that the CMK(Customer Master Key) used to encrypt data can never be retrieved by the user, and the CMK can be rotated for extra security
* Never ever store your secrets in plaintext, especially in your code
* Encrypted secrets can be stored in the code/environment variables
* KMS can only help in encrypting up to 4KB of data per call
* To give access to KMS to someone
	* Make sure the Key Policy allows the user
	* Make sure the IAM Policy allows the API calls

KMS gives us:

* Able to fully manage the keys & policies:
	* Create
	* Rotation policies
	* Disable
	* Enable
* Able to audit key usage(using CloudTrail)
* Three types of Customer Managed Keys(CMK)
	* AWS Managed Service Default CMK: free
	* User Keys created in KMS: **$1/month**
	* User Keys imported(must be 256-bit symmetric key: **$1/month**
* + pay for API keys to KMS($0.03/10000 calls)

## Encryption SDK
* Envelope Encryption for encrypting over 4KB using KMS
* the AWS Encryption helps us to do Envelope Encryption
* Different from S3 Encryption SDK
* Also exists as a CLI tool we can install


**For the exam**: anything over **4KB** of data that needs to be encrypted must use the **Encryption SDK** == **Envelope Encryption** == **GenerateDataKey API**

## SSM Parameter Store
* Secure storage for configuration and secrets
* Optional Seamless Encryption using KMS
* Serverless, scalable, durable, easy SDK, free
* Version tracking of configurations/secrets
* Configuration management using path & IAM
* Notifications with CloudWatch Events
* Integration with CloudFormation

### Parameter Store Hierarchy
* /my-department/
	* my-app/
		* dev/
			* db-url
			* db-password prod/
			* db-url
			* db-password
	* other-app/
* /other-department/

We use this so we can use and take advantage of the `GetParameters` or `GetParametersByPath` APIs

## IAM Best Practices 
### General
* Never use Root Credentials, enable MFA for Root Account
* Grant Least Privilege
	* Each Group / User / Role should only have minimum level of permission it needs
	* Never grant a policy with "*" access to a service
	* Monitor API calls made by a user in CloudTrail(especially denied ones)
* Never ever store IAM key credentials on any machine but a personal computer or on-premise server

### IAM Roles
* EC2 machines should have their own roles
* Lambda functions should have their own roles
* ECS Tasks should have their own roles(ECS_ENABLE_TASK_IAM_ROLE=true)
* CodeBuild should have its own service role
* Create a least-privileged role for any services that requires it
* Create a role per application/lambda function(do not    reuse roles)

### Cross Account Access
* Define an IAM Role for another account to access
* Define which accounts can access this IAM Role
* Use AWS STS(Security Token Service) to retrieve credentials and impersonate the IAM Role you have access to**(AssumRole API)**
* Temporary credentials can be valid between 15 minutes to 1 hour

## Advanced IAM - Authorization Model

### Evaluation of Policies, simplified
1. If there's an explicit DENY, end decision and DENY
2. If there's an ALLOW, end decision with ALLOW
3. Else DENY

### IAM Policies & S3 Bucket Policies
* IAM policies are attached to users, roles, groups
* S3 Bucket Policies are attached to buckets
* When evaluating if an IAM Principal can perform an operation X on a bucket, the **union** of its assigned IAM Policies and S3 Bucket Policies will be evaluated

### Inline vs Managed Policies
* AWS Managed Policy
	* Maintained by AWS
	* Good for power users and administrators
	* Updated in case of new services/new APIs
* Customer Managed Policy
	* Best Practice, re-usable, can be applied to many principals
	* Version Controlled + rollback, central change management
* Inline 
	* Strict one-to-one relationship between policy and principal
	* Policy is deleted if you delete the IAM principal