# IAM + EC2 Notes

## IAM Security
* IAM (Identity and Access Management)
* Your whole AWS Security is there
  * Users: usually a physical person
  * Groups: contains users(ex. engineers, devops)
  * Roles: internal usage within AWS resources
* Root accounts should never be used(and shared)
* Users should be created with proper permissions
* IAM is at the center of AWS
* Policies are written in JSON
* IAM has predefined Managed Policies
* It is best to give users the minimal amount of permissions they need to perform their job(Least privilege principle)

## IAM Federation
* Big enterprises usually integrate their own repository of users with IAM
* This way, one can login into AWS using their company credentials
* Identity Federation uses SAML standard(Active Directory)

## Summary
* One IAM user per Physical Person
* One IAM role per application
* IAM Credentials should never be shared
* Never write IAM credentials in code ever.
* Never use the root account except for initial setup.
* Never use ROOT IAM Credentials

## What's an AMI
* Base images for running EC2 instances
	* Ubuntu
	* Fedora
	* Windows
	* Amazon Linux
* These images can be customized at runtime using EC2 User Data
* We can create custom AMI
	* Pre-installed packages
	* Faster boot time
	* Machine comes configured with monitoring/enterprise software
	* Install your app ahead of time
	* Use someone else's AMI that is optimized for running an app, DB, etc...
* AMI are built for specific AWS Region

## EC2 Instance Characteristics
* Instances have 5 distinct characteristics as advertised:
	* RAM(type, amount, generation)
	* CPU(type, make, frequency, generation, number of cores)
	* I/O(disk performance, EBS optimisation)
	* Network(bandwidth, latency)
	* GPU
* [ec2instances.info](https://ec2instances.info/) can help with summarizing types of instances
* R/C/P/G/H/X/I/F/Z/CR are specialized in RAM, CPU, I/O, Network, GPU
* M are balanced(good at everything but not great at anything particular)
* T2/T3 instance are burstable
	* Burstable eans that overall, has OK CPU performance
	* When the machine needs to process something unexpected, it can burst, and CPU can be very good
	* If the machine bursts, it utilizes "burst credits"
	* If all credits are gone, the CPU becomes BAD
	* If the Machine stops bursting, credits are accumulated over time
* T2 Unlimited
	* you pay extra for having unlimited burst credit balance

## EC2 Pricing
* EC2 Instances prices (per hour) varies based on these parameters:
	* Region you're in
	* Instance types you're using
	* Launch type
	* OS(linux, windows, etc)
* You are billed by the second, with a minimum of 60 seconds
* You also pay for other factors such as storage, data transfer, fixed IP, public addresses, load balancing
* You do not pay for the instance if the instance is stopped

## Launch types
* On Demand
	* short workload, predictable pricing
	* Pay for what we use
	* Has the highest cost but no upfront payment
	* No long term commitment
	* Recommended for short-term an un-interrupted workloads, where you can't predict how the application will behave
* Reserved Instances
	* long workloads (>= 1 year)
	* Up to 75% discount compared to On Demand
	* pay upfront
	* reservation period can be 1 or 3 years
	* reserve a specific instance type
* Convertible Reserved Instances
	* long workloads with flexible instances
	* Same with above but can change the instance type
	* Up to 54% discount
* Scheduled Reserved Instances
	* launch within time window you reserve
* Spot Instances
	* Short workloads
	* Bid for cheap
	* Possibility of losing instances if outbid by other users
	* Spot instances are reclaimed with a 2 minute notification warning when the spot price goes above your bid
	* Used for batch jobs, big data analysis, or workloads that are resilient to failures
	* Up to 90% discount
* Dedicated Instances
	* No other customer will share with the underlying hardware
	* Physical dedicated EC2 server for your use
	* No control of hardware placement and instance placement

* Dedicated Hosts
	* Book an entire physical server
	* Full control of instance placement
	* Visibility into the underlying socket/physical cores of the hardware
	* Allocated for 3 year period reservation
	* Useful for licensing models like BYOL(bring your own license)