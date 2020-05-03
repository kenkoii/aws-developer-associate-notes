# AWS S3
* S3 is one of the main building blocks of AWS
* Infinitely scaling

## Buckets
* S3 allows people to store `objects`(files) in `buckets`(directories)
* Buckets must have a globally unique name
* Buckets are defined at the region level
* Naming Convention
	* No uppercase
	* No underscore
	* 3-63 characters long
	* Not an IP
	* Must start with lowercase letter or number

## Objects
* `Objects`(files) have a `Key`. The key is the **full** path
	* <my_bucket>/my_file.txt
	* <my_bucket>/my_folder1/another_folder/my_file.txt
* There's no concept of "directories within buckets(although the UI will trick you to think otherwise)
* Just keys with very long names that contain slashes `("/")`
* Object Values are the content of the body:
	* Max size is 5TB
	* If uploading more than 5GB, must use "multi-part upload"
* Metadata(list of key/value pairs - system or user metadata)
* Tags(Unicode key/value pair - up to 10) - useful for security/lifecycle
* Version ID(if versioning is enabled)

## S3 Versioning
* You can version your files in AWS S3
* It is enabled at the bucket level
* Same key overwrite will increment the version
* It is best practice to version your buckets
	* Protect against unintended deletes(ability to restore a version)
	* Easily roll back to previous version
* Any file that is not versioned prior to enabling versioning will have version "null"
* If you delete a file, there will be a delete marker instead

## S3 Encryption for Objects
* 4 Methods of encrypting objects in S3
	* SSE-S3: encrypts S3 objects using keys handled & managed by AWS
	* SSE-KMS: leverage AWS Key Management Service ot manage encryption keys
	* SSE-C: when you want to manage your own encryption keys
	* Client Side Encryption

### SSE-S3
* SSE-S3: encryption using keys handled and managed by AWS S3
* Uses S3 Managed Data Key to encrypt
* Object is encrypted server side(S3 side)
* AES-256 encryption type
* Must set header: `"x-amz-server-side-encryption":"AES256"`
* HTTP/HTTPS

### SSE-KMS
* SSE-KMS: encryption using keys handled & managed by KMS
* Uses a KMS Customer Master Key(CMK)
* KMS Advantages: user control + audit trail
* Object is encrypted server side(S3 side)
* Must set header: `"x-amz-server-side-encryption":"aws:kms"`
* HTTP/HTTPS

### SSE-C
* SSE-C: server-side encryption using data keys fully managed by the customer outside of AWS
* S3 does not store the encryption key you provide
* HTTPS must be used
* Encryption key must be provided in the HTTP headers for every HTTP request made
* Header not specified
* S3 throws away the client provided data key right away
* HTTPS only + data key in the header

### Client Side Encryption
* Client library such as the Amazon S3 Encryption client
* Clients must encrypt data themselves before sending to S3
* Clients must decrypt data themselves when retreiving from S3
* Customer full manages the keys and encryption cycle
* Clients are performing encryption and decryption

### Encryption in transit (SSL)
* AWS S3 exposes:
	* HTTP endpoint: non encrypted
	* HTTPS endpoint: encryption in flight
* You're free to use the endpoint you want but HTTPS is recommended
* HTTPS is mandatory for SSE-C
* Encryption in flight is also called SSL/TLS

## S3 Security
* User based
	* IAM policies - which API calls should be allowed for a specific user from IAM console 
* Resource based
	* Bucket Policies - bucket wide rules from the S3 console - allows cross account
	* Object Access Control List(ACL) - finer grain
	* Bucket Access Control List(ACL) - less common

### S3 Bucket Policies
* JSON based policies
	* **Resources**: buckets and objects
	* **Actions**: Set of API to Allow or Deny
	* **Effect**: Allow/Deny
	* **Principal**: The account or user to apply the policy to 
* Use S3 bucket for policy to:
	* Grant public access to the bucket
	* Force objects to be encrypted at upload
	* Grant access to another account(Cross account)

### Other
* Networking:
	* Supports VPC Endpoints(for instances in VPC without www internet)
* Logging and Audit:
	*  S3 Access logs can be stored in other S3 bucket
	*  API calls can be logged in AWS CloudTrail
* User Security:
	* MFA can be required in versioned buckets to delete objects
	* **Signed URLs**: URLs that are valid only for a limited time(ex. Premium video service for logged in users)

## S3 Websites
* S3 can host static websites and have them accessible to anyone on the web
* The website URL will either be:
	* `<bucket-name>.s3-website-<AWS-region>.amazonaws.com`
	* `<bucket-name>.s3-website.<AWS-region>.amazonaws.com`
* If you get a 403(Forbidden) error, make sure the bucket policy allow public reads!

## S3 CORS
* If you request data  from another S3 bucket, you need to enable CORS
* Cross Origin Resource Sharing allows you to limit the number of websites that can request your files in S3(and limit your costs)

## S3 Consistency Model
* **Read after Write** consistency for PUTS of new objects
	* As soon as an object is written, we can retrieve it `(ex. PUT 200 -> GET 200)`
	* This is true, except if we did a GET before to see if the object existed `ex: (GET 404 -> PUT 200 -> GET 404)` - eventually consistent
* **Eventual** consistency for DELETES and PUTS of existing object
	* If we read an object after updating we might get the older version `ex: (PUT 200 -> PUT 200 -> GET 200(might be older version))`
	* If we delete an object, we might still be able to retrieve it for a short time `ex: (DELETE 200 -> GET 200)` 

## S3 Performance
* When you had > 100 TPS(transaction per second), S3 perfromance could degrade
* Behind the scene, each object goes to an S3 partition and for the best performance, we want the highest partition performance
* In the exam, and historically, it was recommended to have 4 random characters in front of your key name to optimise performance
	* `<my_bucket>/5r4d_my_folder/my_file1.txt`
	* `<my_bucket>/a91e_my_folder/my_file2.txt`
* This forces S3 to partition your data correctly
* Never use dates as prefix keys
* Current state:
	* we can scale up to 3500 TPS for PUT and 5500 for GET for each prefix
	* This S3 request rate performance increase removes any previous guidance to randomize object prefixes to achieve faster performance
* Faster upload of large objects **(>=100mb)**, use multipart upload:
	* parallelizes PUTs for greater throughput
	* maximize your network bandwidth and efficiency
	* decrease time to retry in case a part fails
	* Must use multi-part if object size is greater than 5GB
* Use **AWS Cloudfront** to cache S3 objects around the world**(improve reads)**
* **S3 Transfer Acceleration** (uses edge locations) - just need to change the endpoint you write to, not the code.
* If using **SSE-KMS** encryption, you may be limited to your AWS limits for KMS usage(~100s-1000s downloads/uploads per second)

## S3 and Glacier Select
* Note: `Glacier` is another S3 storage tier for long term archival
* If you retrieve data in S3 and Glacier, you may want only a subset of it
* If you retrieve all the data, the network cost may be high!
* With S3 Select/Glacier Select, you can use SQL _SELECT_ queries to let S3 or Glacier know exactly which attributes/filters you want
	* `SELECT * FROM s3objects s where s.Country(nName) like '%United States%'`
* Queries save cost up to 80% and increase performance by up to 400%
* The `SELECT` happens "within" S3 or Glacier
* Works with files in CSV, JSON or Parquet format
* Files can be compressed with GZIP or BZIP2
* No subqueries or Joins are supported
