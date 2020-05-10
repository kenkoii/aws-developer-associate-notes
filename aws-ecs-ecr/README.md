# AWS ECS + ECR

## ECS Clusters
* ECS Clusters are logical grouping of EC2 instances
* EC2 instances run the ECS agent(Docker container)
* The ECS agents registers the instance to the ECS Cluster
* The EC2 instances run a special AMI, made specifically for ECS(ami-ecs-optimized)

## ECS Task Definitions
* Task definitions are metadata in JSON form to tell ECS how to run a Docker Container
* It contains crucial information such as:
	* Image name
	* port bindings
	* memory and cpu required
	* environment variables
	* networking information

## ECS Service
* ECS Services help define how many tasks should run and how they should be run
* They ensure that the number of tasks desired is running across our fleet of EC2 instances
* They can be linked to ELB/NLB/ALB if needed

### ECS Service with Load Balancer
* Dynamic Port Forwarding is a feature of Application Load Balancer
* Ensure Security group of ALB is allowed all traffic access to inbound rules of Security Group of cluster instances
* Host port should be 0 to allow dynamic host port forwarding
	
## ECR
* ECR is a private Docker image repository
* Access is controlled through IAM(permission errors => policy)
* Some commands to push/pull:
	* `$(aws ecr get-login --no-include-email --region eu-west-1)`
	* `docker push 123456.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`
	* `docker pull 123456.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`

## Fargate
* Fargate is Serverless
* No more EC2 to worry about because no need to provision EC2 instances
* Just create task definitions and AWS will run our containers for use
* To scale, just increase the task number

### ECS + X-Ray Integration
* ECS Cluster
	* X-Ray container as daemon
		* x-ray runs on all instances
	* Side Car pattern
		* x-ray runs along with each container as a sidecar
* Fargate Cluster
	* Side Car pattern
		* x-ray runs along with each container as a sidecar 

## ECS + Beanstalk
* You can run Elastic Beanstalk in Single & MultiDocker Container mode
* Multi Docker helps run muiltiple containers per EC2 in EB
* This will create for you:
	* ECS Cluster
	* EC2 instances, configure to use the ECS Cluster
	* Load Balancer(in high availability mode)
	* Task Definitions and execution
* Requires a config file **Dockerrun.aws.json** at the root of source code
* Your Docker images must be pre-built and stored in ECR for example

## Summary
ECS is used to run Docker containers and has 3 flavors:

### ECS Classic
Provision EC2 instances to run containers onto

* EC2 instance must be created
* We must configure the file **/etc/ecs/ecs.config** with the cluster name
* The EC2 instance must run an ECS Agent
* EC2 instances can run multiple containers on the same type:
	* You must **not** specify a host port(only container port)
	* You should use an ALB with the dynamic port mapping feature
	* The EC2 instance security group must allow traffic from the ALB on all ports
* ECS tasks can have IAM Roles to execute actions against AWS
* Security Groups operate at the instance level, not task level

### ECR
* ECR is tightly integrated with IAM
* Some commands:
	* `$(aws ecr get-login --no-include-email --region eu-west-1)`
	* `docker push 123456.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`
	* `docker pull 123456.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`
* `aws ecr get-login` generates a "docker login" command
* **In case an EC2 instance (or you) cannot pull a Docker image, check IAM**

### Fargate
* ECS Serverless, no more EC2 to provision
* AWS provisions containers for us and assigns them ENI
* Fargate containers are provisioned by the container spec(CPU/RAM)
* Fargate tasks can have IAM Roles to execute actions against AWS

### ECS Integrations
* ECS does integrate with X-Ray
	* To make it work, X-Ray must be running as a 2nd container within the task definition
	* You can use the ready-to-use AWS image
* ECS does integrate with CloudWatch Logs:
	* You need to setup logging at the task definition level
	* Each container will have a different log stream

### CLI to know
* Obtain a docker login command against ECR
	* `$(aws ecr get-login --no-include-email --region eu-west-1)`
* Push
	* `docker push 123456.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`
* Pull
	* `docker pull 123456.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`
* Create a service on ECS
	* `aws ecs create-service`
* Build Docker image
	* `docker build -t demo .`


### EKS
Managed Kubernetes service by AWS