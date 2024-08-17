## This hands-on solving Final Project: 
# BlogDjango
BlogDjango

## Description

The TechPro Education Blog Page Application aims to deploy blog application as a web application written Django Framework on AWS Cloud Infrastructure. This infrastructure has Application Load Balancer with Auto Scaling Group of Elastic Compute Cloud (EC2) Instances and Relational Database Service (RDS) on defined VPC. Also, The Cloudfront and Route 53 services are located in front of the architecture and manage the traffic in secure. User is able to upload pictures and videos on own blog page and these are kept on S3 Bucket. This architecture will be created by Firms DevOps Guy.

## Problem Statement

- Your company has recently ended up a project that aims to serve as Blog web application on isolated VPC environment. You and your colleagues have started to work on the project. Your Developer team has developed the application and you are going to deploy the app in production environment.

- Application is coded by TechPro Education Fullstack development team and given you as DevOps team. App allows users to write their own blog page to whom user registration data should be kept in separate MySQL database in AWS RDS service and pictures or videos should be kept in S3 bucket. The object list of S3 Bucket containing movies and videos is recorded on DynamoDB table. 

- The web application will be deployed using Django framework.

- The Web Application should be accessible via web browser from anywhere in secure.

- You are requested to push your program to the project repository on the Github. You are going to pull it into the webservers in the production environment on AWS Cloud. 

In the architecture, you can configure your infrastructure using the followings,

# Blog Project Tasks

## Infrastructure Setup

1. **Create VPC and all components**
   - Create VPC
   - Create and attach an internet gateway
   - Create Subnets
   - Create Route Tables
   - Create Endpoint

2. **Create Security Groups (ALB ---> EC2 ---> RDS)**
   - ALB Security Group
   - EC2 Security Groups
   - RDS Security Groups
   - NAT Instance Security Group

3. **Create RDS**
   - Create a subnet group for our custom VPC
   - Create Database
     - Database engine: `MySQL`
     - Version: `8.0.35`

4. **Create two S3 Bucket** 
5. **GITHUB**
6. **Create NAT Instance**
7. **Create Target Group**
8. **Create Application Load Balancer**
9. **Create IAM roles**
10. **Launch Template Configuration**
11. **Create Auto Scaling Group**
12. **Create CloudFront Distribution**
13. **Creating fail-over routing policies**
14. **Create a Lambda Function and Trigger Event**


## PART 1  -----------------------------> **Create VPC and all components**  ----------------------------
### STEP 1: Create VPC

- First, go to the VPC and select Your VPC section from the left-hand menu, click create VPC.

- choose VPC Only

- create a vpc named 'vpcFP' with 10.0.0.0/16 CIDR

```
    - IPv4 CIDR manual input
    - no ipv6 CIDR block
    - tenancy: default
```
- click create

- `IMPORTANT`:  enable DNS hostnames for the vpc 'vpcFP'

  - Select 'vpcFP' on VPC console ----> Actions ----> Edit VPc Settings ----> DNS settings, Enable DNS hostnames
  - Click save 

### STEP 2: Create an internet gateway named 'final-project-ig'
- Go to the Internet Gateways from left hand menu

- Click Create Internet Gateway
   - Name Tag "final-project-ig" 
   - Click create button

-  Attach the internet gateway 'final-project-ig' to the vpc 'vpcFP'
```   
    - Actions ---> attach to VPC
    - Select VPC named "vpcFP"
    - Push "Attach Internet gateway"
```

### STEP 3: Create Subnets
- Go to the Subnets from left hand menu
- Push create subnet button


1. 
Name tag          :fp-subnet-public-1a
VPC               :vpcFP
Availability Zone :us-east-1a
IPv4 CIDR block   :10.0.1.0/24

2. 
Name tag          :fp-private-subnet-1a
VPC               :vpcFP
Availability Zone :us-east-1a
IPv4 CIDR block   :10.0.11.0/24

3. 
Name tag          :fp-subnet-public-1b
VPC               :vpcFP
Availability Zone :us-east-1b
IPv4 CIDR block   :10.0.2.0/24

4. 
Name tag          :fp-private-subnet-1b
VPC               :vpcFP
Availability Zone :us-east-1b
IPv4 CIDR block   :10.0.12.0/24


- `IMPORTANT`: enable Auto-Assign Public IPv4 Address for public subnets

- Go to the Subnets from left hand menu

  - Select 'fp-subnet-public-1a' subnet ---> Action ---> Edit subnet settings
  ---> select 'Enable auto-assign public IPv4 address' ---> Save

  - Select 'fp-subnet-public-1b' subnet ---> Action ---> Edit subnet settings
  ---> select 'Enable auto-assign public IPv4 address' ---> Save

### STEP 4: Route Tables

- Go to the Route Tables from left hand menu
- push the create route table button

- create a private route table (not allowing access to the internet) 
  - name: 'fp-private-rt'
  - VPC : 'vpcFP'
  - click create button


- click Subnet association button >> Edit subnet associations and show the route table `fp-private-rt` with private subnets

- Click Edit subnet association
- select private subnets;
  - fp-private-subnet-1a
  - fp-private-subnet-1b
  - and click save

- Create a public route table that allows internet access

- push the create route table button
  - name: 'fp-public-rt'
  - VPC : 'vpcFP'
  - click create button


- click Subnet association button >>> Edit subnet associationsand show the route table 
- Click Edit subnet association

- select public subnets;
  - fp-subnet-public-1a
  - fp-subnet-public-1b
  - and click save

- select Routes on the sub-section of fp-public-rt

- click edit routes
- click add route
- add a route
    - destination ------> 0.0.0.0/0 (any network, any host)
    - As target;
      - Select Internet Gateway
      - Select 'final-project-ig'
      - save routes 


### STEP 5: Create Endpoint
- Connect to S3 via Endpoint

- click `Create Endpoint`
```text
Name: fp-s3-endpoint
Service Category : AWS services
Service Name     : com.amazonaws.us-east-1.s3(Gateway)
VPC              : vpcFP
Route Table      : fp-private-rt
```
- Create Endpoint



## PART 2  -----------------------------> **Create Security Groups (ALB ---> EC2 ---> RDS)**  ----------------------------
### STEP 1 - Creating Security Groups

1. Create security group for ALB
Name: fp-alb-sg
Inbound: Type: HTTP Port: 80 Source: Anywhere IPv4
         Type: HTTPS Port: 443 Source: Anywhere IPv4

2. Create security group for NAT Instances
Name: fp-nat-instance-sg
Inbound: Type: All traffic Source: vpcFP
         Type: SSH Port: 22 Source: Anywhere IPv4

3. Create security group for private Instances
Name: fp-ec2-sg
Inbound: Type: HTTP Port: 80 Source: from fp-alb-sg
         Type: HTTPS Port: 443 Source: from fp-alb-sg
         Type: SSH Port: 22 Source: vpcFP

4. Create security group for RDS
Name: fp-rds-sg
Inbound: Type: Mysql/Aurora Port: 3306 Source: fp-ec2-sg



## PART 3  -----------------------------> **Create RDS**  ----------------------------
### STEP 1 - Creating Subnet groups for RDS

- click to 'Create DB subnet group'
- in 'Subnet group details':
  Name: fp-rds-subnet-g
  Description: fp-rds-subnet-g
  choose 'vpcFP'
- in 'Add subnets':
Availability Zones choose:
  us-east-1a and us-east-1b
subnets:
  choose 2 private subnets

### STEP 2 - Creating RDS Instance

- Select Standard create as method,
- Engine: MySQL
- Version: 8.0.35
- Templates: Free Tier (For high availability we may choose production, when selected options viewed also change)
Settings
- DB instance identifier: wordpress1
- Master username: admin
- Master password: MyProje2024
- Confirm password: MyProje2024
Instance Configuration
- DB Instance class: Burstable classes db.t3.micro
switch include previous generation classes button to see t3.micro
Storage
- general purpose SSD (gp2)
- 20 GB
- Deselect Enable storage autoscaling
Connectivity
VPC
- vpcFP
Subnet Group
- fp-rds-subnet-g
- Public access: No
VPC security group
Choose existing
- fp-rds-sg
Availability Zone
- az1a
Additional configuration
- Database port: 3306
Database authentication
- Password authentication
Additional configuration
- Initial database name: wp
Backup
- Uncheck Enable automated backups
Encryption
- Uncheck
Monitoring
- Uncheck
Maintenance
- Uncheck Enable auto minor version upgrade
Maintenance window
- No preference

Click Create database



## PART 4 ------------------------------>  Create two S3 Bucket   <-----------------------
### STEP 1
- Go to the S3 service on AWS console
- Create a bucket of `pf-s3-kendiadin` with following properties, 

```bash

Versioning                  : Disabled
Server access logging       : Disabled
Tagging                     : 0 Tags
Object-level logging        : Disabled
Default encryption          : Default
CloudWatch request metrics  : Disabled
Object lock                 : Disabled
Block all public access     : unchecked
```

### STEP 2: Create a S3 as Static WebSite :

 1. Create Static WebSite / "www.[your sub-domain name]"
 
  - Go to S3 service and create a bucket with sub-domain name: "www.[your sub-domain name]"
  - Public Access "Enabled"
  - Upload Files named "index.html" and "error.jpg"
  - Permissions>>> Bucket Policy >>> Paste bucket Policy
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "xxxxxxxx/*"
        }
    ]
}
```
  - Properties>>> Set Static Web Site >>> Enable >>> Index document : index.html 
   
## PART 5 ---------------------------------->   GITHUB   <-----------------------------
### STEP 1 Create new repo 
- go on gitHub on browser -> login 
- left-hand menu click `New` 
- Repository name: aws-final-project
- Public: checked as default 
- click `Create repository`

### STEP 2 **Copy files downloaded or cloned from Techproeducation repo on Github**
- Create a file "teach files" and open terminal in the file
- do:
``` bash
git clone https://github.com/techproeducation-batchs/AD279-AWS-DEVOPS.git
```
- go to -> AWS -> 35-AWS-Final Project (Project-6): `and you will see all files you need FOR FINAL PROJECT`

### STEP 2 Push your files in your public repo
- click your `aws-final-projec` and copy your https link from code
- https like: https://github.com/your-user-name/aws-final-project.git 
- go to your desktop and open `Terminal` in Desktop
- Clone your repo on local:  (you have to copy your https link and paste)
``` bash
 git clone https://github.com/your-user-name/aws-final-project.git 
 ```
- You see your file on Desktop as like: aws-final-project
- Open it and copy files from final project which given by teacher  in `35-AWS-Final Project (Project-6)` file
- Open `Terminal` in aws-final-project and command next code:
``` bash
git add .
git commit -m "Files for project added"
git push
``` 
Now your files are ready on your remote public repo

### STEP 3 Change your data (All data which must be changed are ready let's do it!)
### STEP 3a
- open your repo on gitHub 
- go to `BlogDjango` -> src -> .env
- click `edit` change: PASSWORD = your_rds_password
- click `commit changes...` -> "password saved" -> click `commit`
### STEP 3b
- go to `cblog` -> settings.py -> click `edit this file` -> go to row: 85 

you will see (enter your: NAME, USER, HOST):    
```py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'your_db_name', # database name in RDS is written here
        'USER': 'admin', # database master username in RDS is written here 
        'PASSWORD': config('PASSWORD'), # FinalProject2024!
        'HOST': 'your_endpoint',  # database endpoint is written here
        'PORT': '3306' # database port is written here
    }
```
- then go to row: 147

```py
AWS_STORAGE_BUCKET_NAME = 'your_bucket_name_which_will_stor_your_post_data' # please enter your s3 bucket name
```
- click `commit changes...` -> "data for rds and s3" -> click `commit`

- ---------------------Your repo is ready to use!


## PART 6 ---------------------------------->  Create NAT Instance  <------------------------
### STEP 1 Launch ec2
 create an instance from AMIs designed as NAT.

- Go to EC2 Menu Using AWS Console
ami-037eb9e678c1a8ed9

AMI             : ami-037eb9e678c1a8ed9 (Nat Instance)
- include keypair
- Network settings: 
VPC - vpcFP 
subnet: fp-subnet-public-1a
- Select Security Group: fp-nat-instance-sg

- Select created Nat Instance on EC2 list
- Tab Actions Menu ----> Networking ----> Change Source/Destination Check ---> stop, Disable

### STEP 2 Configuring the Route Table

- Go to Route Table and select "fp-private-rt"

- Add Route
```
Destination     : 0.0.0.0/0
Target ----> Instance ----> Nat Instance
```
WARNING!!! ---> Please do first "NAT instance" for "Private Blog Web Page EC2".

## PART 7 ---------------------------------->  Create Target Group  <-----------------------

- Go to `Target Groups` section under the Load Balancing part on left-hand menu and select `Target Group`

- Click `Create Target Group` button

  - Basic configuration

    Choose a target type    : Instances
    Give Target Groups Name : fp-target-group
    Protocol                : HTTP
    Port                    : 80
    VPC                     : vpcFP

  - Health checks

    Health check protocol   : HTTP
    Health check path       : /

  - Advance health check settings

    Port                    : Traffic port
    Healthy threshold       : 3
    Unhealthy threshold     : 2
    Timeout                 : 5 seconds
    Interval                : 10 seconds
    Success codes           : 200

  - Tags

    ```text
    Key                     : Name
    Value                   : fp-target-group
    ```

- Click next

-  Do not register any instances into the target group.

- Click `Create Target Group` button.


## PART 8 ---------------------------------->  Create Application Load Balancer  <-------------------

Go to the Load Balancing section on left-hand menu and select `Load Balancers`.

- Click `Create Load Balancer` tab.

- Select the `Application Load Balancer` option.

- Step 1: Configure Load Balancer

```text
Name            : fp-alb

Scheme: Internet-facing
IP address type: IPv4

- Step 2: Network Mapping

Choose vpcFP
Availability Zones          : Choose all AZ's

- Step 3: Configure Security Groups

- Select an existing  >>>> security group.

Name            :  fp-alb-sg
  

- Step 4: Listeners and Routing: 
A listener is a process that checks for connection requests, using the protocol and port that you configured.

Load Balancer Protocol      : HTTP
Load Balancer Port          : 80
Default action Forward to: Select your Target Group
!!! MyTargetGroup that we created for Application Load Balancer. It will be same both for Application Load Balancer and Auto Scaling

- Step 5:
Add-on services             : Keep it as default

- Step 6:

Tags                        :
    - Key   : Name
    - Value : fp-alb
```

- Review and if everything is ok, click the `Create load balancer` button.

- Please wait for changing the state from `provisioning` to `active`.





## PART 9 ---------------------------------->  Create IAM roles  <--------------------------------
### STEP 1 Create IAM role to reach S3 from "Private WEB EC2"

- Go to IAM Service from AWS console and select roles on left hand pane
- click create role
Role ---> create role ---> AWS service ---> Use case EC2 ---> Next ---> Policy ---> "AmazonS3FullAccess" ---> Next 
Role Name : pf-ec2role-for-s3
Role description: my S3 Full Access for Endpoint
click create button

### STEP 2: Create IAM Role for LAMBDA:

- Go to `Roles` on the left hand menu and click `create role`.

Type of Trusted Entity      : AWS Service
Use Case                    : Lambda
Permissions                 : AmazonS3FullAccess 
Role Name : pf-lambdarole-for-s3
click create button



## PART 10 ---------------------------------->  Launch Template Configuration  <-----------------------

- Launch template name            : fp-template
- Template version description    : fp-template

- Amazon Machine Image (AMI)
  Quick Start: 
  Ubuntu: server 22.04
- Instance Type: t2.micro

- Key Pair: your key pair

- Network settings
subnet: fp-private-subnet-1a

- Security groups
Please select security group named >>> fp-ec2-sg

- Storage (volumes)
keep it as default (Volume 1 (AMI Root) (8 GiB, EBS, General purpose SSD (gp2)))

- Resource tags
Key             : Name
Value           : blog web page
Resource type   : Instance

- Advanced details >>> 
- IAM instance profile: select your IAM role: fp-ec2role-for-s3
User data
- Please paste the script below into the `user data` field.

```bash
#!/bin/bash
apt-get update -y
apt-get install git -y
apt-get install python3 -y
cd /home/ubuntu/
git clone https://github.com/gammar93/aws-final-project.git
cd /home/ubuntu/aws-final-project/BlogDjango
apt install python3-pip -y
apt-get install python3.8-dev default-libmysqlclient-dev -y
apt-get install libjpeg-dev zlib1g-dev -y -qq
pip3 install -r requirements.txt
cd /home/ubuntu/aws-final-project/BlogDjango/src
python3 manage.py collectstatic --noinput
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py runserver 0.0.0.0:80
```
- click `Create Launch Template`


## PART 11 ---------------------------------->  Create Auto Scaling Group <----------------------------
  note: Create an Auto Scaling Group that keeps the target group in initial size

- EC2 AWS Management console, select Auto Scaling Group from the left-hand menu and then click Create Auto Scaling Group
Name: fp-auto-scaling
Step 1: Choose launch template or launch configuration:

- Switch to the `Launch template`, select newly created launch template and click `Next` to continue.

Step 2: Configure settings:
- Network

```text
VPC         : vpcFP
Subnets     : Select all private Subnets
```

Step 3: Configure advanced options:

- Select  >>>> `Attach to an existing load balancer` option

- Click Choose from your load balancer target groups and find Target Group: `MyTargetGroup`

- Leave VPC Lattice as is (No VPC Lattice)

- Health Checks

EC2 Health checks is always enabled
Additional Health Check type           : Click Turn on ELB health checks (Recommended)
Health check grace period   : 200 seconds

- Additional Settings : Keep it as default

Step 4: Configure group size and scaling policies:

- Group size

Desired capacity        : 2
Minimum capacity        : 1
Maximum capacity        : 3

- Scaling policies : No scaling policies

- Instance scale-in protection: Do not check

Step 5: Add notifications:

- Skip Notification

Step 6: Add Tags

Key     : Name
Value:  : fp-auto-scaling
Step 7: Review and create Auto Scaling Group.

- `Recommendation`: You can check if your webpage is working using `ALB DNS name`

## PART 12 ---------------------------------->  Create CloudFront Distribution  <-------------------------
- Go to CloudFront service and click "Create a CloudFront Distribution"

- Create Distribution :
  - Origin:
    - Origin Domain: EC2 Public IPv4 DNS
    - Protoco : HTTP only
  - Default Cache Behavior:
    - Compress objects automatically :yes
    - Viewer Protocol Policy: Select "Redirect HTTP to HTTPS"
    - Allowed HTTP methods : GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
    - Cache key and origin requests : Cache policy and origin request policy (recommended)
    - Cache policy : cachingDisabil
    - Origin request policy - optional : AllViewer
  - Settings
    - Alternate Domain Names (CNAME): [your-domain-name],   add www.
    - Custom SSL Certificate: Select your newly created certificate

- Leave the other settings as default.

- Click "Create Distribution".

- It may take some time distribution to be deployed. (Check status of distribution to be "Enabled")

- When it is deployed, copy the "Domain name" of the distribution. 

- `Recommendation`: You can check if your webpage is working using `CloudFront DNS name`


## PART 13 ---------------------------------->  Creating fail-over routing policies  <----------------------
### STEP 1 : Create health check for "N. Virginia" instance

- Go to left hand pane on route_53 and click the Health check menu 
- Click Health check button
- Configure Health Check

```bash 
1. Name: fp_healthcheck
What to monitor     : Endpoint
Specify endpoint by : Domain name
Protocol            : www.example.com
Port                : 80
Path                : leave it as /
Advanced Configuration 
Request Interval    :  Standard (20seconds)
Failure Threshold   : 2
Explain Response Time:

Failure = Time Interval * Threshold. If its a standard 20 seconds check then 2 checks is actually equal to 40 seconds. So be careful of how these two different settings interact each other.

String Matching     : No 
Latency Graphs:     : Keep it as is
Invert Health Check Status: Keep it as is
Disable Health Check: Keep it as is
Explain: If you disable a health check, Route 53 considers the status of the health check to always be healthy. If you configured DNS failover, Route 53 continues to route traffic to the corresponding resources. If you want to stop routing traffic to a resource, change the value of Invert health check status.

Health Checker Regions: Keep it as default
click Next
Get Notification   : None

click create and wait that the status is unhealthy approximately after 40 seconds the instance healthcheck will turned into the "healthy" from "unhealthy"
```
### STEP 2: Create A record for  "N. Virginia" instance IP - Primary record

- Got to the hosted zone and select the public hosted zone of our domain name
- Click create record
- select "Failover" as a routing policy
- click next

```bash
Record Name :"www"
Record Type : A
TTL:"60"
Value/Route traffic to : 
  - "Ip address or another value depending on the record type"
    - enter IP IP address of N.Virginia 
Routing: "Failover"
Failover record type    : Primary
Health check            : fp_healthcheck
Record ID               : Failover Scenario-primary
```
- Click `Create record`

### STEP 3: Create A record for S3 website endpoint - Secondary record

- Click create record
- select "Failover" as a routing policy
```bash
Record Name :"www"
Record Type : A
TTL:"60"
Value/Route traffic to : 
  - "Alias to S3 website endpoint"
  - N.Virginia(us-east-1)
  - Choose your S3 bucket named "www.[your sub-domain name].net"
Routing: "Failover" 
Failover record type    : Secondary
Health check            : keep it as is
Record ID               : Failover Scenario-secondary
```
- Click `Create record`

## PART 14 ---------------------------------->  Create a Lambda Function and Trigger Event   <---------------

### STEP 1: Create Lambda Function

- Go to Lambda Service on AWS Console

- Functions ----> Create Lambda function

1. Select Author from scratch
   -Author from scratch
  Name: fp-lambda
  Runtime: Python 3.12
  Change default execution role :
  Execution role ->    Use an existing role
  Role: pf-lambdarole-for-s3
  click->create function

### STEP 2: Setting Trigger Event

- Go to Configuration sub-menu and click AddTrigger on Diagram  

Add trigger
Trigger Configuration : S3
- Bucket              : fpbucketgamer
- Event Type          : All object create events
- Recursive invocation         : checked
click `Add`

### STEP 4: Create Function Code

- Go to the Function Code sub-menu and paste >>> >>>> code seen below:

```python
import json
import boto3

def lambda_handler(event, context):
    s3 = boto3.client("s3")
    
    if event:
        print("Event: ", event)
        filename = str(event['Records'][0]['s3']['object']['key'])
        timestamp = str(event['Records'][0]['eventTime'])
        event_name = str(event['Records'][0]['eventName']).split(':')[0][6:]
        
        filename1 = filename.split('/')
        filename2 = filename1[-1]
        
        dynamo_db = boto3.resource('dynamodb')
        dynamoTable = dynamo_db.Table('change me!!!!!')
        
        dynamoTable.put_item(Item = {
            'id': filename2,
            'timestamp': timestamp,
            'Event': event_name,
        })
        
    return "Lambda success"
```

- Click `DEPLOY` button


#### Congratulations!