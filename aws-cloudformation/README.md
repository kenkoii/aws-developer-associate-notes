# CloudFormation
Managing your infrastructure as code

## Overview
* Avoid manual work as it is prone to mistakes
* Manual work will be very tough to reproduce
	* In another region
	* In another AWS Account
	* Within the same region if everything was deleted

## What is CloudFormation
* **CloudFormation** is a declarative way of outlining your AWS Infrastructure, for any resources(most of them are supported)
* Given a declarative template, CloudFormation creates those resources for you, in the right order, with the exact configuration that you specify

### Benefits
* Infrastructure as code
	* No resources are manually created
	* The code can be version controlled using git
	* Changed to the infrastructure are reviewed through code
* Cost
	* Each resources within the stack is stagged with an identifier so you can easily see how much a stack cost you
	* You can estimate cost of your resources using the template
	* Savings strategy
* Productivity
	* Ability to destroy and re-create an infrastructure on the cloud on the fly
	* Automated generation of Diagram for your templates
	* Declarative programming
* Separation of concerns: create many stacks for many apps and many layers:
	* VPC stacks
	* Network stacks
	* App stacks
* Don't re-invent the wheel
	* leverage existing templates on the web
	* leverage the documentation

### How CloudFormation Works
* Templates have to be uploaded in S3 and then referenced in CloudFormation
* To update a template, we can't edit previous ones. We have to reupload a new version of the template to AWS
* Stacks are identified by a name
* Deleting a stack deletes every single artifact that was created by CloudFormation.

### Deploying CloudFormation templates
* Manual way:
	* Editing templates in the CloudFormation Designer
	* Using the console to input parameters, etc
* Automated way:
	* Editing templates in YAML file
	* Using the AWS CLI to deploy the templates
	* Recommended way when you want to fully automate your flow

## Building Blocks
Template components:

* Resources: your AWS resources declared in the template
* Parameters: dynamic inputs for your template
* Mappings: the static variables for your template
* Outputs: References to what has been created
* Conditionals: List of conditions to perform resource creation
* Metadata

Template helpers:

1. References
2. Functions

## Resources
* **Resources** are the core of CloudFormation templates(MANDATORY)
* They represent the different AWS Components that will be created and configured
* Resources are declared and can reference each other
* AWS figures out creation, updates and deletes of resources for us
* Resource types are of the form `AWS::aws-product-name::data-type-name`

## Parameters
* **Parameters** are a way to provide inputs to your AWS CloudFormation template
* They're important to know if:
	* You want to reuse your templates across the company
	* Some inputs can not be determined ahead of time
* Parameters are extremely powerful, controlled, and can prevent errors from happening in your templatesa thanks to types

### When should you use a parameter?
* Is this CloudFormation likely to change in the future?
* You won't have to reupload a template to change its content

### How to Reference a Parameter
* The `Fn::Ref` function can be leveraged to reference parameters
* Can be used anywhere in a template
* The shorthand for this in YAML is `!Ref`

## Mappings
* **Mappings** are fixed variables within your CloudFormation Template
* Differentiate different envs (dev vs prod), regions, ami type., etc
* All the values are hardcoded within the template
* `Fn::FindInMap`
	* `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ` 

### Mappings vs Parameter
* Mappings are great when you know in advance all the values that can be taken and that they can be deduced from variables such as
	* Region
	* Availability Zone
	* AWS Account
	* Environment(dev vs prod)
	* Etc.
* The allow safer control over the template
* Use parameters when the values are really user specific and you dont know it in advance

## Outputs
* Outputs section declares *optional* outputs values that we can import into other stacks(if you export them first)!
* You can also view the outputs in the AWS Console or in using the AWS CLI
* It's the best way to perform some collaoration cross stack, as you let expert handle their own part of the stack
* You can't delete a CloudFormation Stack if its outputs are being referenced by another CloudFormation stack
* Use `Fn::ImportValue` or `!ImportValue` to import an `Output`

## Conditions
* Conditions are used to control the creation of resources or outputs based on a condition
* Each condition can reference another condition, parameter value or mapping
* Conditions:
	* `DoSomething: !Equals [ !Ref EnvType, prod ]` 
	* Fn::And
	* Fn::Equals
	* Fn::If
	* Fn::Not
	* Fn::Or

## Intrinsic Functions
* **Fn::Ref** function can be leveraged to reference
	* Parameters => returns the value of the parameter
	* Resources => returns the physical ID of the underlying resource
* **Fn::GetAtt** get attributes of a certain resource
	* Look at documentation to know the attributes
	* `!GetAtt Resource.Property`
* **Fn::FindInMap** to return a named value from a specific key
* **Fn::ImportValue** to import exported values from other stacks
* **Fn::Join** join values with a delimiter
	* `!Join [ ":", [ a, b, c ] ]`
* **Fn::Sub** allows you to substitute values within a string
	* String must contain `${VariableName}` 
	* `!Sub String`

## Rollbacks
* Stack Creation Fails:
	* Default: everything rolls back(gets deleted)
	* Option to disable rollback and troubleshoot what happened
* Stack Update Fails
	* The stack automatically rolls back to the previous known working state
	* Ability to see in the log what happened and error messages