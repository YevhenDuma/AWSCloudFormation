#MyBB forum (http://www.mybb.com/)  - https://github.com/YevhenDuma/AWSCloudFormation/tree/master/MyBB 
	URL to access: http://forum.yevhen.net
Cloud Formation templates that creates infrastructure for MyBB forum, installs MyBB and scale up/down if needed.

I've created 4 Cloud Formation templates, mainly by service name, because:
- it's easier to work with infrastructure divided;
- it's easier for understanding (in overall there will be more that 1k lines in one file);
- it's easier for changes.

So next 4 CloudFormation 
- Core.template  
- SNS.template
- RDS.template  
- EC2.template 

#Core.template
This template creates core infrastructure:
- Virtual private cloud (VPC);
- 4 subnets: two for application (in different Az) and two for database (to enable Multi Az);
- Networking (Including routing, internet gateway, network ACL's),

As result in output there will be:
- VPC ID;
- 4 Subnets ID.

Assumpptions:
- For RDS there is no internet gateway defined, used separate routing table
- Since instance needs to have internet access (to access yum repo, to download application, to download database if needed), I've decided to assign external IP per each instance. As possible solution is to use: 1) NAT gateway, but I haven't found that it's already defined in Cloud Formation or 2) NAT instance, but it's additional costs, possible bottle net (yes, it's better to have two, but additional costs). But access is restricted per IP or CIDR that is configurable.

#SNS.template
This template creates SNS topic. Later it will be used in RDS and EC2 to alert in case of any problems.

#RDS.template
This template creates MySQL database in RDS service.
First you need to pass parameters defined in Core.template and SNS.template:
- RDS subnets
- VPC
- SNS topic

Parameters like DB name, DB size, DB username, DB password are configurable.

As result in output will be hostname.

Assumptions:
- Access to instance only from VPC CIDR. There is possibility to allow access from particular security group, but as RDS will be created before EC2, on this step there will be no security group ID. As a solution - keep RDS and EC2 in one file.
- Configured monitoring and alarms: for CPU usage (if more than 70), for Low Disk Space (Low than 1Gb);
- Maybe in future to use ssl between client and server (http://dev.mysql.com/doc/refman/5.7/en/creating-ssl-files-using-openssl.html).

#EC2.template
This template:
- Creates LoadBalancer and saves it DNS name to output;
- Creates Launch configuration, which starts instance of some type (it's configurable), download MyBB from official site (URL is also configurable), configure apache httpd and MyBB (it takes DB info from parameters), and populates database from predefined file, if database if empty;
- Creates alarms and scale up/down when needed;
- Adds monitoring to each instance, for Memory, status check; 
- Load to CloudWatch 2 files (httpd access log and MyBB httpd error log) and defines alerts (example: if amount of 404 errors is greater than 2 over 2 minutes);
- Creates Security groups (example: Access by 80 port is allowed from ELB only, access by port 22 is allowed by exact CIDR, which is configurable.

This templates allows update, so it's possible to do deployments without downtime.

As I am working as DevOps last few years, I was trying to create not only secure solution with lot's of monitoring, etc, but easy to understand for developers Iaas, and solution that allows to have easy deployments (in this case just update template, change URL to application. In best scenation: rpm/deb package (I was thinking about that, but it will take much more time). Configuration is done via Cloud Formation Init and parameters. Configuration file is stored in /etc/mybb and linked to mybb publishing directory. So as result there will be forum up and running, but not installation steps.

Missing:
- Monitoring for ELB. My goal was not to create all possible monitorings, but to show examples how to monitor (full list of supported monitors: http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/elb-metricscollected.html )
- Memory monitoring is configured on AWS side, but not inside instance. Disk monitoring is missing.

Assumptions:
 - I am not satisfied with security, I know things that I need to improve, but it's related to time/financial;
- I was trying to achive working solution, so anyone who is using MyBB can take my templates and have Iaas;
- Instead of S3 bucket could be used any repo/file storage.

MyBB config:
Admin user: admin
Password: 12321q

#Feedback:
- Interesting task, that really check your AWS knowledge
- But this task takes a lot of time, since in most cases you are updating something, less - creating something new.
