Steps to create environments:
Run next command to configure aws credentials

	aws configure

To create Test environment, run next commands from folder with templates and parameter files

1) To create core infastructure:

AWSCFNAME="Core"

AWSENV="Test"

aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json

2) To create Route53 hosted zone:

AWSCFNAME="Route53"

AWSENV="Test"

aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json

3) To create SNS topic:

AWSCFNAME="SNS"

AWSENV="Test"

aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json

4) Create Role with static content:

AWSCFNAME="WebStatic"

AWSENV="Test"

aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json --capabilities CAPABILITY_IAM


5) Create Web Applicaton role:

AWSCFNAME="WebApp"

AWSENV="Test"

aws cloudformation create-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json --capabilities CAPABILITY_IAM


update stack:
aws cloudformation update-stack --stack-name ${AWSCFNAME}-${AWSENV} --template-body file://./${AWSCFNAME}-companyNews.json --parameters file://./${AWSCFNAME}-companyNews-${AWSENV}-parameters.json --capabilities CAPABILITY_IAM



http://webstatic-elasticl-11grobhya2oig-612325766.eu-west-1.elb.amazonaws.com/static/images/logo.png

Update template files


Time to scale:
Static - 3 min and 8 sec