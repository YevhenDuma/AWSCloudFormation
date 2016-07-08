Table of Contents
======

[About](https://github.com/YevhenDuma/AWSCloudFormation/tree/master/companyNews#about)

[Architectual diagram](https://github.com/YevhenDuma/AWSCloudFormation/tree/master/companyNews#architectual-map)


[Assumptions](https://github.com/YevhenDuma/AWSCloudFormation/tree/master/companyNews#assumptions)


[Decisions](https://github.com/YevhenDuma/AWSCloudFormation/tree/master/companyNews#decisions)


[Expected result](http://github.com/YevhenDuma/AWSCloudFormation/tree/master/companyNews#expected-result)


[Result](https://github.com/YevhenDuma/AWSCloudFormation/tree/master/companyNews#result)




About
======

CloudFormation templates to create 2 environments (Test and Production) for Java based application (without RDBMS).


Architectual map:
======

![alt text](https://github.com/YevhenDuma/AWSCloudFormation/blob/master/companyNews/ArchDiagram.png "companyNews Architectual diagram")


Assumptions:
======

1. All environments should be equal

2. Consider monitoring and auto recovery

3. Consider autoscaling


Decisions:
=======

- One of easiest ways to deploy application is to use cloud.
- AWS provides automation tool - CloudFormation, to create infrastructure from code
- With CLoudFormation we can use docker (ECS service) or linux instances (EC2). In case of big Java application and need to increase in a moment docker will be better solution.
- To install needed packages, configure application we can use Puppet, Chef or Ansible. In this example to simplify solution none of Configuration Management tool was used. Instead installation and configuration done via UserData. For different environment different values could be stored in parameter files
- With CloudFormation there is possibility for 0-downtime deployment
- It's bad to deploy application without keeping in mind whole infrastructure. So as result here is Core infrastructure templates. I've also used Route53 to have nice DNS names.
- All templates are separated. It's possible to have everything in one file, or to have dependend CloudFormation templates, but in some cases you need to updated CloudFormation templates independently
- nGinx is pretty good for static files and works as http proxy
- For https it is possible to use Amazon Certificate Management service
- Scaling is part of templates. If CPU reaches XX % during YY time AWS will start one more instance. Maximum amoun of instances could be different for different environments
- There is possibility to have Scheduled scaling


Expected result:
======

After uploading this CloudFormation templates you will have whole infrastructure up and running, and Web Application accesible from some limited CIDR or to anyone if needed.


Result:
======

URL to access: **http://DOMAINNAME/companyNews**

Time to scale: around 4 minutes


Steps to create environments:
------

- Run next command to configure aws credentials


	`aws configure`

- First you need to create Core infrastructure, and check output in CloudFormation from AWS console. After for each environment update WebApp-companyNews-ENV-parameters.json file.

- Procedure to create Test and Production environment is same, next will be steps for Test environment. For Production instead of  AWSENV="Test" use  AWSENV="Production"

To create Test environment, run next commands from folder with templates and parameter files:

1. To create Core infastructure:

	`AWSCFNAME="Core"`

	`AWSENV="Test"`

	`aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json`

2. To create Route53 hosted zone:

	`AWSCFNAME="Route53"`

	`AWSENV="Test"`

	`aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json`

3. To create SNS topic:

	`AWSCFNAME="SNS"`

	`AWSENV="Test"`

	`aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json`

4. Deploy Web Appication

	`AWSCFNAME="WebApp"`

	`AWSENV="Test"`

	`aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json --capabilities CAPABILITY`IAM`


if update needed (e.g. different url or some additional steps in deployments) simply run:

	`aws cloudformation update-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json --capabilities CAPABILITY`IAM`
