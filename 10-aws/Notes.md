**1. Explian how will you design a highly available and scalable multi-tier app.**
As a firt step-> Create **VPC** [I will interact with developers to understand number of appl's that are going to be deployed / number of applications that are goign to us ethis VPC]
depending upon that, I will define a **CIDR** block for the VPC.
-> within VPC, i will create **public** and **private** subnets and also define **CIDR range for both public and provate subnets** accordingly.
-> with in public subnet, I will place load balancer
-> with in private subnet, I will deploy virtual machines and these virtual machines will be configured through **AWS ASG(Auto scaling group)**, so that these VM's can be scalable.
   with in **VM's I will deploy application server where application will be deployed**. These applications on VM's interact with **DB** which I am going to deploy on the private subnet.
   if required, I will create a saperate private subnet for the DB, it totally depends on the level of isolation/ I can use a managed DB instance according to the requirement from the deveploers.  
   
  Now, after created VPC as explianed above,
  any **external user** can access -> **load balancer** (will also be grated access to the private subnets- here load balancer will be a public/private subnet VPC load balancer), this load balancer assigned with target groups / ASG (as VM's in the private subnet) ->
  request fro Load balancer **goes to the application server in the private subnet**,**from where application interacts with DB fetch the require information and returns information back to the client.** 

  Note: avaliability and scalability here is achieved here using AWS ASG.
        security by defining right size of CIDR block and application and DB defined in the private subnets / if required I will use managed AWS DB servuces like RDS.
   <img width="302" height="430" alt="image" src="https://github.com/user-attachments/assets/0b9c1af9-d7d4-4b69-a708-ed40763293da" />

   
**2. What is AWS NAT(Network Address Translation) and when is it used?**
In my current company, we have VM which as application running in the prvate subnet, where these backend application need to download some dependencies from GITHUB.
because thes applications deployed in the private subnet. Obviously they don't have external access or access to the internet. So in these cases, to allow the applications talk to internet or talk to GitHub to download packages,
we create a Nat gateway. So Nat in AWS performs network address translation. So we create a Nat gateway and create a route in AWS where the route will have a route table, which has the destination route
0.0.0.0 pointing to the Nat gateway instead of internet gateway.

**Application / any process in the application tries to download something from internet -> request sent to -> NAT, performs Network Address Translation-> through NAT request sent to GITHUB.**

Note: NAT will change the source IP address (application server IP) to the NAT public IP and sends request to the destination address i.e here it is GITHUB.

This way source IP of the application is not exposed.
Client request -> OSI Layer -> L4 (client request is converted into packets, packet will have source IP, Dest IP and all other information, so thet your packet actually know how to travel from where to where)-> here NAT will change same packet where client request is converted into packet at L4,change application source IP to NAT's public IP and send request to the public internet. here 
destination will not respond to the source application server IP instead forwards it to the NAT public IP. This way Application Server IP in the private subnet is not exposed to the external world.

**3. How to enable Internet access to the application deployed in a private subnet of VPC?**
To enable secure external access to application:
-  I will configure NAT gateway
-  create new route in AWS -> Route Table which will have destination route 0.0.0.0/0 as NAT gateway
-  NAT will peform SNAT (source IP network address translation) and sends application server request to the destination IP in the internet.
<img width="1038" height="232" alt="image" src="https://github.com/user-attachments/assets/da92efc9-53e4-4a9a-8b6d-f3d451c70849" />


**4. Can applications in different subnets of a VPC interact by default, if no, why?**
So the answer to this question is absolutely yes. So by default application within different subnets of the VPC can interact with each other. Or you can log in to one of the instance and ping instance in any other subnet saying that they are in the same VPC.

when you create VPC -> A default route table created and lets say, for example, a subnet CIDR is 10.0.0.0/16 **to local** -> That means by default, different subnets, different applications within the VPC should interact with each other. This is the default behavior of AWS. And this is the default behavior of many other cloud providers as well. Because by default, VPC is virtual, private cloud and communication should be enabled within the virtual private cloud. Any external communication should be disabled.
we can block this type of deafult communication between subnets using AWS security Group / NACL's but AWS by deafult configuration it allows applications to talk to each other irrespective of the subnets they are in.
<img width="1772" height="523" alt="image" src="https://github.com/user-attachments/assets/13d5da49-aa1e-4532-b310-c0909ad37eb0" />


**5. Explian NACL vs SG and which one do you use in your organization?**
when you create a VPC in AWS, by default, the route rule of VPC allows communication between subnets of AWS.
That means-> So although you have applications in different subnet or instances in different subnet, they can still talk to each other because of the default route rule of AWS.

Now to restrict access in some cases.
For example, in our current organization we have a subnet for database. And we want to restrict access to this subnet from other subnets. So in this case we have used an NACL through which we have restricted access to this subnet. 

So the default behavior of NACL is to deny any traffic at the subnet level. So you can define rules in NACL where you can define if the traffic has to be allowed on a specific port (or) if you want to deny traffic on a specific port (or) you can also define deny all rule in NACL, which denies all the traffic on all the ports of that particular subnet.
**So this is one use case where we have used NaCl.**

**However, in all the cases you might not want to use NaCl. You might want to define rules at the instance level** -> Then you use **Security Groups**
So there are cases where different applications within a subnet in our organization needs different traffic rules.
For example, on one of the application we want port 80 to be exposed. So that application should be reached on port 80 by applications in the other subnet (Or) sometimes even external access should be allowed (or) Whereas on other instances we don't want this port 80 to be exposed. **So when we want to define rules at the instance level, not at the subnet level, We go for security groups**, 

**and majority of the times we go for combination of NaCl and security group as well.**
For example, where we are using NaCl + security group in our organization. 
NACL allowing port 80 at subnet level + except allowing port 80 on instance level which has application supported on port 80 and all other instances be disabled at instance level using security groups.

NACL is stateless in nature. Both ingress and egress need to be defined in traffic rules.
ex: if you allow ingress to port 80 the you will have to allow egress.

Security Group is statefull in nature. If ingress is allowed by default egress is also allowed.
ex: if you allow ingress to port 80 y default egress is also allowed.

Note: ingress (inbound) egress (outbound)


**6. EC2 instance terminated unexpectedly, How will you troubleshoot?**
Steps:
- Using AWS cloud trail, we can find out what can be the potential issue and I will look at API call called TeminateInstance- this will give us information, if that termination happend because of ASG/ automation script/ any cycle policy impacting the EC2 Instance.
- one thing I will also check is, if that instance is spot instance (usually spot instances is not recommended to use for critical applications because spot instance can be terminated any time). 


**7. Lambda function fails randomly, how will you fix the issue?**
Lambda function can be any programming language usually performs HTTP request / interacts with any AWS services like RDS/s3/EBS etc., to perform some activities lambda function is invoked, when lambda is invoked, it fails randomly and there are no errors in the log. 
Steps of troubleshooting:
- I will enable tracing for the lambda function and it can be enabled using AWS x-ray service. 
- Through x-ray traces I understand if there is any tall latency - betweeen lamdba function interacting with any services and why latency.
- if latency at EBS, will check in which region / AZ is EBS.
- I will try increase retries. instead of starting Lambda function after fail, I will try including try/catch/exception logic with in the function. 
- as a final step I will try increasing the timeout if we see genuine latency.
- I will monitor this entire process for sometime - if isue is not sorted will go and rewrite lambda function maybe  choosing different programming language / do something to reduce the latency of the request.


**8. What will you do when AWS RDS storage is full?**
- increase RDS storage size which is instant solution to the problem to unblock the users - first take snapshot (for safer side)/ increase storage /  use AWS ASG .
- as a long term fix, I will head to RDS and check DB / Tables / objects used then will identify which of these are using large space. ex: I will run sql query to identify which DB service is consuming the large space.
<img width="1014" height="52" alt="image" src="https://github.com/user-attachments/assets/da294ea4-7fee-45ab-ae2a-f3fdc2a468ef" />
- Will share this information with the developers / DB admin and understand why any of that particular service is taking large space and will ask the concerned time to debug and fix the issue.
- to aviod failures, I will also head to AWS Cloud watch and create a metric and monitor for AWS RDS storage. At cloud watch I can enable metric called FreeStorageSpace - this will inform when RDS storage is about to be full. This will help me know when storage is about to be full for me to take action when issue is still not fixed.


**9. Developer Deleted Critical Resources like S3, RDS and EC2. What will you do?**
Steps:
- I will frist try to recreate the instances - fortunetly mot of these services have backup mechanism. Ex: s3 (option avaliable for versioning - goback to previous version and create s3 from previus version- atleast people working on it are not blocked) 
- RDS as point-in-time restore solution - I will restore RDS instance
- EC2 instance has snapshots - I will retrieve volumes from snapshopts and AMI to create instance.

For longterm solution:
I will work on strict RBAC / IAM - Alway good approach to follow zero pervilage - just to provide right people to access the resources.
- within the organisation, I will take a look at IAM user policies attached to the IAM users - will start working on reducing the previlages / eventually reaching to least privilages where developers dont have delete options and anything that they dont require. 

devops engineers by majority should be the administrators and they should be performing such activities and even we should not do it manually - will look at IaC and performed through Terraform/ Cloud Formation template and through versioning control system like GIT. 


**10. Explian a cost optimization activty that you performed in the current org?**
Steps:
- cost optimization is part of my day-to-day job roles. As a devops engineer I was expected to take care of cost optimization task.
-  Recently I worked on a cloud cost optimization task where the task was to identify unused EBS volumes and delete those unused EBS volumes.
backstory for it: A lot of our developers create EBS volumes. They also request us to create EBS volumes. They use the EBS volumes. At times the EBS volumes remain unused. And this has been happening over the period of time. So for last one year or last two years, there have been bunch of EBS volumes piled up. And some of these EBS volumes are attached to AWS. Some of them are already taken as snapshots and some of them are completely unused. So the activity that is assigned to me is to identify such unused EBS volumes and delete those volumes.

Example 1:  run awscli -> run aws EC2 describe volumes to identify the list of volumes. But then I realized the better approach is to use lambda functions. So I moved to a lambda function where I used Python as a scripting language. I tried to identify so within the Python code I tried to look at unused volumes. So I have used boto3 for it and using Boto3 I got the list of unused volumes and I deleted using Boto3 itself. And I have set up this lambda function periodically so that I don't have to remember that I need to
run this lambda function. This will be executed every fourth Friday of the month. So this way I have reduced significant cost for the organization. 
Example 2:  Again on EBS itself, I worked on, EBS GP2 GP3 two types. One is GP2 - is an older version and you know this also comes up with high cost. Whereas GP3 - comes with a low cost and it also comes with high IOPs. we have volumes that are created for the last two years and even older. So I wrote a lambda function to look at the EBS volumes, which are with GP2, and I upgraded those things to GP3 with a single Python command. And for this as well, I have used Boto3. So Boto3 is a simple framework that you can use with Python, and this will do the heavy lifting. You don't have to write a lot of code. You just need to use the right modules in Boto3. And those modules will help you interact with EBS. For example, in Boto3 there is S3 client, there is EC2 client. You will use those clients and interact with the components.

So this is how I took care of cloud cost optimization tasks.


**11. Explian a recent challenge that you faced with AWS and how did you slove it?**
Steps:
-  AWS CodeCommit->In the recent times, a major challenge that I faced is with AWS Codecommit. So recently AWS announced the deprecation of AWS code commit end of life for it, and we are majorly impacted with AWS Codecommit. So we have almost 200 repositories. All our source code is hosted on AWS Codecommit because we are heavily dependent on the AWS DevOps services. So we were using AWS Codepipeline. We were using code deploy, code commit, and we have invested a lot on it. Like I said, 200 repositories or 200 microservices source code is hosted on AWS code commit. Now this announcement came as a surprise for it for us. So what we decided is to identify a migration strategy. So we have to move out of AWS Codecommit.

- GitHub - We evaluated GitHub enterprise. We evaluated GitLab and multiple other solutions. We want to ensure that we move to a permanent thing this time, something that will not be end of life. So we decided to move to GitHub enterprise. So I led this effort in first thing comparing the different version control systems, hosted version control systems that I evaluated GitHub, GitLab, Bitbucket. I found out GitHub is more promising solution these days.

* One is because of the ecosystem the features that it is adding.
* Also with respect to AI features that it is adding support for Copilot and also GitHub enterprise supports a lot of features, including project management.

So I decided we can move to GitHub and I have designed a strategy.

- Dashborad-So I prepared a dashboard where within the dashboard I have identified less critical repositories. Critical repositories and most important repositories. So I have categorized repositories as per this. And I have designed the migration strategy. In first one month we will migrate the less critical ones implement the ci CD through AWS Codepipeline. We are still sticking to that. But integration of Codepipeline to GitHub. And then the next month we will migrate the critical ones. And finally the most important repositories. So this way I worked on this entire migration strategy for almost four months and with zero impact We have migrated all the repositories to GitHub enterprise.

I have also received appreciation for this migration activity because I have led the entire effort, and I have also reported to the stakeholders through the dashboard. So with zero downtime, I have migrated everything to GitHub enterprise.


**12. Auto Sclaing Group Not Launching EC2, what can be the issue?**
So there is a launch template and the launch template was creating EC2 instances. But right now it is no longer creating the instances. What can be the problem?
Steps:-
- launch configuration: will start looking into the template that is assigned to the Auto Scaling group, or the launch template launch configuration with the Auto Scaling Group, and will start troubleshooting.
- valid AMI and instance type: I will understand what is written in this launch configuration - see valid AMI and instance type used in the launch configuration and the both exist in the region that you are trying to create.
- Subnet, AZ: in the next step I will try to see if there is any subnet and is there any issue with respect to capacity or if there is any issue with the availability zone?

say if no problem with above

-EC2 Instance quota: there is a possibility that you are hitting EC2 resource quota. Or maybe during that particular time, you cannot create any more EC2 instances in that region or availability zone.
-Spot Instances: it can be that you are using spot instances and spot instances offer encounter interruption. That can also be one of the reasons.
- Finally, I will try to see if it is any permissions related issue. It is quite possible that the IAM role that is assigned to the auto scaling group has permissions issue. So previously it was working. Maybe IAM role has a policy attached which used to have the required permissions and right now someone must have modified the policy. Maybe while working on some regular maintenance activity, some access was removed to the auto scaling group because of which it no longer can create resources, specifically EC2 on the AWS.


**13. Which AWS services do you use in your day to day life?**
Obviously in AWS you have 200 plus services and each service solves a different problem. The use cases are different, and you might know some 40 to 50 services as DevOps engineer.
- look at job description and you will know what services interviewers are expecting from you like VPC, EC2, EKS, ECR, S3 then you should be talking about these services.
- you can say, As part of my DevOps and cloud role in the current organization. I use various AWS services. It totally depends on the request from the developers. However, some of the services that I use at frequent basis are VPC for networking related things, EC2 for compute, X for managed Kubernetes, ECR for pushing the container images, S3 bucket for storing the objects.

- Similarly, you can just say the services that align with this - For example, RDS. For our databases we use EBS for volumes. We also use AWS Codepipeline for CI, CD, and code deploy for deployment. So these are the frequent services that I use.
**Of course, you know this is according to this job description. If the job description has different services talk about those services.**


**14. Have you used AWS EFS? if yes, what issues did you run into?**
-  EFS is elastic file storage, and it is useful in the cases of shared storage. If we go for EBS, EBS is a block storage, whereas EFS using NFS can be shared between multiple EC2 instances.
- So when you are looking for a shared storage that is scalable and that is also serverless in nature, then EFS is the right solution available on AWS. In our case, we use EFS because we have lot of data that is shared across EC2 instances, some configuration files, or some data that has to be mounted to different EC2 instances. So for such concurrent data retrieval and concurrent usage, we use AWS Elastic File Storage or Block Storage.
Now coming to the issue recently I ran into an issue with EFS, so while working for one of the development teams, I came across the mount target unreachable issue with EFS. So basically I was trying to mount the EFS, but mount was failing. So I started to look into the issue, and first thing that I did was to understand if the EFS mount, because EFS is a serverless solution. I tried to see if the mount target is available in all the availability zones where the EC2 instances are deployed. We have EC2 instances in different availability zones US East 1A 1B 1C.
*I checked first if EFS has the mount target in all the availability zones.
* I try to look at the security group attached to the EFS. we work in a highly secure environment. We create security groups and we assign the ports to the resources through the security groups. I mean, we allow the traffic through the security group. So I looked at the security group to identify if port 249 is allowed in the inbound traffic.
* finally, I try to see the DNS resolution and IAM that is attached to the EFS access point. I finally realized the issue was with the IAM assigned to the EFS access point.

I updated the IAM policy and everything worked fine. So the issue with EFS mount target unreachable was solved.


**15. When will you go for EFS over EBS in realtime?**
EFS is elastic File system-> This is used when you are looking for a shared file system. So when you have multiple EC2 instances and all these instances should share a common file system to retrieve the data, or maybe to concurrently access the data, then you go for EFS.
Elastic block storage -> is dedicated to a single EC2 instance, maybe for high performance, then you are looking for EBS.

EBS is a block storage that comes with high performance, whereas EFS is a shared file system that allows concurrent read and write of data to the file system.

FOr example: you are using a Kubernetes cluster with multiple nodes and pods deployed on those nodes. Let's say you are working on a image processing application where each pod across different nodes access the images, process the image, and they upload the image onto a common file system. In this case, because it's a common file system and different pods are uploading images to the file system, maybe at the same time as well. You cannot go for EBS, but you should go for EFS.
In a nutshell, when multiple pods in the Kubernetes cluster are trying to access the common file system, then you go for EFS.

Whereas when you're working with databases where you need high performance and the volume is directly attached to the database instance so that you have high throughput and you have, you know, very fast read and write operations, then you go for EBS, which is a block storage where the instance directly has access to the block or the storage.
EFS is highly scalable
EBS is used for high prformance. Basically useful for the db relasted applications.


**16. How to disable AWS console access to the IAM users?**
As a DevOps engineer, you have 3 predominant ways of creating the users:
*use the CLI
* you can use infrastructure as code, maybe Terraform,
* or you can also do it using console.

If you do it through console, it's straightforward. 
<img width="1716" height="494" alt="image" src="https://github.com/user-attachments/assets/16a14cfe-6548-4fe5-8a94-89db6a669cbf" />

If you are going for Terraform in Terraform there is a module for IAM and when you use this module to create the user, just make sure you use the field to disable the console access.


**17. How can AWS lambda function in one AWS account connect to S3 bucket in other account?**
<img width="226" height="108" alt="image" src="https://github.com/user-attachments/assets/f8405d7e-394e-419f-9da5-1024d09d87ef" />
But very honestly, there is not much difference if the resources are in same AWS account or different AWS accounts. Specifically, if we talk about this question in AWS account, there is a Lambda function.
- First step, If the Lambda function does not exist, just create the Lambda function.
- Second step, Head to the lambda execution role. Head to the policy attached to the role and just provide the name of the S3 bucket. You don't have to provide the account in which S3 bucket exists, because S3 buckets are universal in nature.
<img width="490" height="196" alt="image" src="https://github.com/user-attachments/assets/45cedc3b-da04-478a-8847-24d3082fe251" />
- As the third step you will head to S3 bucket, this time in the AWS account you will head to S3 bucket bucket policy for every S3 bucket a policy is attached. Just head to that policy and make sure AWS Lambda function within account A has access to it.
So even if you go to AWS Lambda function in account A, go to the execution role of Lambda and copy ARN of it and paste it in the bucket policy. Your job is done.
For example this is a sample bucket policy for you. This is just for your understanding.
<img width="828" height="348" alt="image" src="https://github.com/user-attachments/assets/6c278ccc-4b7a-49dd-a6ce-2781baaec604" />


**18. What is AWS STS and how does it work?**
STS is a service that stands for Security Token Service. If a service user or principal wants to get temporary security credentials of an IAM role, AWS STS services used. So typically this service allows the services, users or principals to assume an IAM role and get temporary security credentials so that it can perform the activity which IAM role is capable of.

For example, let's say there is a lambda function, and this lambda function wants to get information from a DynamoDB.
So lambda function wants to get the data from the DynamoDB.
Let's say Lambda function is not granted with the access, but this Lambda function during its execution wants to temporarily get the access to the IAM role, which has permissions to get items from the DynamoDB.
In this case, within the Lambda function, a DevOps engineer can use the STS service, and using the STS service, DevOps engineer can assume the IAM role, get the temporary security credentials from the IAM role, which will allow the Lambda function to access the items of DynamoDB.

<img width="1214" height="766" alt="image" src="https://github.com/user-attachments/assets/db8f56b2-4dd9-4389-b0ba-a10c9ecf4c96" />

So this is a Lambda function that a DevOps engineer might write. This solves the same problem as I was explaining. So Lambda function gets the access to the DynamoDB through an IAM role. So DevOps engineer is using Python boto3 which is a very popular Python module. So using Boto3 DevOps engineer is using the STS service. So got access to the STS client. Now you can see assume role. So using STS DevOps engineer is assuming this particular IAM role or Lambda function is assuming this particular IAM role.
And once assume role is complete, this assume role will have temporary security credentials. So the same thing access ID security access key and session token is read from the credentials or is read from the assume role. Now this lambda function will have access to the DynamoDB temporarily. You can also define the time a particular time frame for this temporary security credentials. So this way a lambda function can get or assume IAM role that is in the same AWS account, 

or 

it can be in a different AWS account as well. If it is in a different AWS account, along with the IAM role should also be updated with the trust policy.


**19. What is trust policy in AWS and why is it used?**
AWS allows services, users or principals to temporarily assume an IAM role and perform the activity on their behalf.
Let's say there is an IAM role which can fetch data or items from DynamoDB. So if there is a service in AWS that wants to temporarily use this IAM role and get the details from DynamoDB, this service can assume the IAM role using AWS STS.

However, because every service should not assume the IAM role and every service or user should not get the information from DynamoDB. Within the IAM role, we will define a trust policy and this trust policy tells the IAM role. Also tells the STS service who can assume the IAM role. So you can define only particular service. Maybe Lambda service can assume this IAM role and fetch the information from DynamoDB. Temporarily.This can be for 30 minutes. This can be for specific period of time.So this is what trust policy does.

For example: During the execution of this lambda function, it temporarily wants to assume this IAM role, which can fetch information from DynamoDB only during the execution of the Lambda function. It wants to assume this IAM role using STS and It wants to fetch some items from DynamoDB.

In this case, within this IAM role. We can go to the trust policy, and within the trust policy we will provide the information of the Lambda function. Now this IAM role can be in the same AWS account. This DynamoDB can be in the same AWS account or it can be in a different AWS account as well. Trust policy works across the accounts also.
<img width="1780" height="786" alt="image" src="https://github.com/user-attachments/assets/68bcbac5-25f7-4d33-a3a9-e53c6acf3277" />
So within the trust policy you can say this dev console role can be assumed by lambda function.
Or 
this can be assumed by EC2.
Or 
this can be assumed by any service within the AWS.
It doesn't have to be a service/It can be IAM User as well.

So this is how trust policies work in AWS.


**20. How a Lambda function in AWS Account A interact with Dynamodb in Account B?**
- As step one I will head to AWS account B and within AWS account B I will create an IAM role, and this role will have a policy attached and this policy will have access to DynamoDB Get items.
- As step two, I will create a Lambda function in AWS account A, and within the Lambda function I will use AWS STS (security token service) to fetch temporary security credentials of role that is created in account B. Of course, by default this will be blocked.
- I will head to the AWS account B again. And within the AWS account B IAM role, I will head to the trust policy, and within the trust policy, I will grant access to the Lambda function in AWS account A so within the trust policy. Access to AWS Lambda function is granted, so I will only grant access to a particular Lambda function that wants to interact with the DynamoDB.

Now following these steps, access will be granted across the accounts.
Example code snippet
<img width="1230" height="632" alt="image" src="https://github.com/user-attachments/assets/c59beab0-b2a4-479f-a356-4a102c67652c" />
paste here 
<img width="1766" height="740" alt="image" src="https://github.com/user-attachments/assets/200a4081-d0be-4a59-b1ea-31d597976e0d" />




