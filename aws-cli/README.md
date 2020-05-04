# AWS CLI

## S3
* List buckets: `aws s3 ls`
* List contents of bucket/directory: `aws s3 ls <S3Uri>`
* Make bucket: `aws s3 mb <bucket-name>`
* Remove bucket: `aws s3 rb <bucket-name>`
* Copy object: 
	* `aws s3 cp <S3Uri> <S3Uri>`
	* `aws s3 cp <LocalPath> <S3Uri>`
	* `aws s3 cp <S3Uri> <LocalPath>`
* Move object:
	* `aws s3 mv <S3Uri> <S3Uri>`
	* `aws s3 mv <LocalPath> <S3Uri>`
	* `aws s3 mv <S3Uri> <LocalPath>`
* Delete object: `aws s3 rm <S3Uri>`
* Sync directories and S3 prefixes:
	* `aws s3 sync <S3Uri> <S3Uri>`
	* `aws s3 sync <LocalPath> <S3Uri>`
	* `aws s3 sync <S3Uri> <LocalPath>`

	
## EC2

### Running AWS CLI on EC2
* Better way is using IAM Roles
	* attach IAM Role to EC2 instances
	* Specify in policy what EC2 instances should be able to do (least privilege principle)
	* EC2 instances can then use these profiles automatically without any additional configurations
	* This is the best practice on AWS and you should 100% do this

### EC2 Instance Metadata
* Allows AWS EC2 instances to "learn about themselves" without using an IAM Role for that purpose
* The URL is `http://169.254.169.254/latest/meta-data`
* You can retreive the IAM Role name from the metadata, but you cannot retrieve the IAM Policy
* Metadata = Info about the EC2 instance
* Userdata = launch script of the EC2 instance

	
## STS

### Decoding Encoded error messages
* `aws sts decode-authorization-message --encoded-message <message>`
* This requires the `DecodeAuthorizationMessage` policy attached to the caller's role


## AWS SDK Credentials Security
* It's recommended to use the default credential provider chain
* The default credential provider chain works seamlessly with:
	* AWS credentials at ~/.aws/credentials (only on our computers or on premise)
	* Instance Profile Credentials using IAM Roles (for EC2 machines, etc...)
	* Environment variables **(AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)**
* Never ever store AWS credentials in your code
* Best practice is for credentials to be inherited from mechanisms 100% IAM Roles if working from within AWS Services

### Exponential Backoff
* Any API that fails because of too many calls need to be retried with Exponential Backoff
* These apply to rate limited API
* Retry mechanism included in SDK API calls
* Retry time doubles after each failed attempt 

## AWS CLI Profiles
* `aws configure --profile <profile-name>`
* Running this should create another entry in the `~/.aws/config` and `~/.aws/credentials` file
* To target the other profile when running aws commands:
	* `aws s3 ls --profile <profile-name>`
