# Twoge

Welcome to the official repository for Twoge, a niche social media platform dedicated to Doge, envisioned by Elon Musk. As a team member of Codeplatoon, this README serves as your guide to deploying and maintaining the Twoge application on AWS.

## About Twoge

Twoge is a unique platform exclusively for sharing and enjoying content about Doge. With Twoge, users can post, interact, and celebrate everything related to their favorite automobile brand, Doge. The platform aims to connect Doge enthusiasts worldwide and be the central hub for Doge discourse.

## Table of Contents

- [Deployment Overview](#deployment-overview)

- [Step-By-Step Instruction Guide](#step-by-step-instruction-guide)

## Deployment Overview

This project uses various AWS services for a robust and scalable deployment. Below is a summary of the services and their role in Twoge:

- **AWS EC2**: Hosts the Twoge application servers.
- **AWS S3**: Stores static assets like images and videos.
- **AWS IAM**: Manages permissions and access to AWS resources.
- **AWS VPC**: Provides a secure, isolated network for Twoge.
- **AWS ALB**: Balances traffic across EC2 instances.
- **AWS ASG**: Manages scaling of EC2 instances based on demand.
- **AWS SNS**: Sends notifications regarding the application's health.
- **AWS RDS**: Optionally, hosts the database in a more secure setup.

## Step-By-Step Instruction Guide

### 1. Create a VPC with two public subnets.
* Open the Amazon VPC console. 
* In the **VPC Dashboard**, choose **Create VPC**.
* Under **VPC settings**, choose **VPC only**.
* Complete these fields as follows:
   - The IPv4 CIDR block should be ```10.0.0.0/24```.
   - The rest is default.
* Enable DNS hostnames and resolution:
   - Select VPC and click Actions.
   - Click Edit VPC settings.
   - Enable DNS hostnames and DNS resolution.
* Subnet1
   - Choose VPC ID that you just created.
   - Subnet name: twoge-pub-2a.
   - Availability Zone: us-west-2a.
   - IPv4 VPC CIDR block: 10.0.0.0/25.
* Subnet2
   - Choose VPC ID that you just created.
   - Subnet name: twoge-pub-2b.
   - Availability Zone: us-west-2b.
   - IPv4 VPC CIDR block: 10.0.0.128/25.
NOTE: [Visual subnet calculator](https://www.davidc.net/sites/default/subnets/subnets.html)
* Create Internat Gateway and Attach to VPC
   - Name: twoge-igw-prd.
   - Click "Create internet gateway".
   - Attach to VPC- xin-twoge-vpc-uw2.
   - Click "Attach internat gateway".
* Create route table
   - Name: twogee-rt-prd.
   - VPC is xin-twoge-vpc-uw2.
   - Go to Routes tab and Click Edit routes.
   - Add route: 
   - Destination: 0.0.0.0/0 .
   - Target: Internet Gateway- twoge-igw-prd.
* Subnet Associate with route table
   - Go to Subnet associations.
   - Add twoge-pub-2a and twoge-pub-2a and save it.

### 2. Create an IAM role for S3 access.
   - Go to the IAM dashboard.
   - Create Role: Click Create role > AWS service > EC2.
   - Permission: Select AmazonS3FullAccess and attach it.
   - Give a name(TwogeS3AccessRole-xin) and description and create role.

### 3. Host the static files on an S3 bucket.
   - Create Bucket.
   - Name: xin-twoge-s3-uw2.
   - Region: us-west-2.
   - Uncheck-Block all public access.
   - Keep the default setting.
   - Choose Permissions > Under Bucket Policy > choose Edit. To grant public read access for your website, copy the following bucket policy, and paste it in the Bucket policy editor.
  
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
               "arn:aws:s3:::Bucket-Name",
               "arn:aws:s3:::Bucket-Name/*"
                
            ]
        }
    ]
}
```
  - Upload the files or folder.
  - Test the list of S3 bucket .
```sh
aws s3 ls s3://xin-twoge-s3-uw2
```

### 4. EC2

* Lauch an instance:
  - Name:ntwoge-uw2-prd
  - Amazon Machine Image > Linux 2 AMI
  - Instance type: t2.micro
  - Select your key pair
  - Networking setting: 
```sh
VPC: you created VPC for twoge
Subnet: choose your public subnet
Auto-assign public ip: Enabled
Create security group: ntwoge-ec2-sg
```
* Inbound Security Group Rules:
  - HTTP | 80 | 0.0.0.0/0
  - HTTPS | 443 | 0.0.0.0/0
  - SSH | 22 | 0.0.0.0/0
  - PostgreSQL | 0.0.0.0/0
  - Custom TCP | 9876 | 0.0.0.0/0
* User data: 
```sh
#!/bin/bash
sudo yum update -y
sudo yum install git -y
git clone https://github.com/chandradeoarya/twoge
cd twoge
sudo yum install git python3-pip -y
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt

echo 'SQLALCHEMY_DATABASE_URI = "postgresql://username:password@endpoint/database"' > .env

echo '
Description=Gunicorn instance to serve twoge

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
WorkingDirectory=/home/ec2-user/twoge
Environment="PATH=/home/ec2-user/twoge/venv/bin"
ExecStart=/home/ec2-user/twoge/venv/bin/gunicorn app:app -c /home/ec2-user/twoge/gunicorn_config.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target' > twoge.service

sudo cp twoge.service /etc/systemd/system/twoge.service
sudo systemctl daemon-reload
sudo systemctl enable --now twoge
sudo systemctl status twoge
```

### 5. Deploying RDS

* Create Database
  - Go to the AWS Management Console.
  - Search bar select RDS service.
  - Click Create database.
* Configuration
  - method choose Standard create.
  - Engine type > select PostgreSQL.
  - Engine Version > choose 15.3-r2.
  - Templates > select Free tier.
* Settings
  - DB instance identifier: Create and enter the name of your database-dbinstance.
  - Master username: It's recommended to change the username-postgrestwoge.
  - Master Password: Enter a personal password and confirm the master pwd.
* Instance configuration
  - keep the default value.
  - Choose db.t3.micro.
* Storage
  - Leave the default values for Storage.
  - Storage autoscaling > deselect Enable Storage autoscaling.
* Connectivity
  - Select Don't connect to an EC2 compute resource.
  - For Virtual private cloud, choose the same VPC selected for your EC2.
  - Public Access > Yes.
  - Create new VPC security group > Name: twoge-rdc-sg.
* Additional Configuration
  - Database port to 5432.
  - Database authentication > select Password authentication
* Monitoring
  - default value
* Additional Configuration
  - Initial database name > enter the database name associated with your Flask app-tdb.
* Create Database
  - Click Create Database 
  ( Note: the initialization of the database will take about 5-10 minutes.)
* Edit Inbound
  - Once the database is created, navigate to the Connectivity & security tab.
  - Select the VPC Security groups.
  - rules:
```sh
Type: PostgreSQL
Protocol: TCP
Port Range: 5432
Source: Custom
***Destination: Get the security group of your EC2 instance previously created.(Enable an Amazon EC2 instance to communicate with the RDS database)
```

### 6. Create an image of the instance
* Select the instance in the EC2 dashboard.
* Navigate to Actions > Image and templates > Create Image > Enter an image name and add a description.
* Keep the default value.
* Create image.

### 7. Create Launch Template using the Image created

* In the navigation pane, choose Launch Templates, then click Create launch template.
* Provide a name and description for the launch template.
*In the Application and OS images, click on My AMIs. 
*Select Owned by me and choose Amazon Machine Image (AMI) that you created.
* Set the Instance Type to t2.micro.
* Configure additional parameters such as:
  - key pair: select one used when creating EC2.
  - Network Settings: leave the subnet as default.
  - Firewall/Security group: select existing security group created for ec2 instance
  - Advanced details: IAM instance profile -> Select The IAM role created that allows EC2 have access to the S3 bucket.
* Click the Create launch template button.

### 8. Creating a Target group

* Navigate to Target Groups. In the navigation pane, under Load Balancing, click on Target Groups.
* Click on the Create target group button.
* Choose Target Type: For Target type, select Instances (since you’re routing traffic to EC2 instances).
* Set Target Group Details:
  - Name: Enter a name for your target group.
  - Port: Enter 9876 as the port on which the targets receive traffic.
  - Select the VPC that was used for the EC2 instance.
  - Create target group

### 9. Create Load Balancers
* Navigate to Load Balancers in the EC2 dashboard.
* Create Load Balancer
   - Click on 'Create load balancer'.
   - Choose 'Application Load Balancer'.
* Enter a unique name for your Load Balancer.
* VPC- Select the VPC that was used for your EC2 instance
* In the 'Mappings' section, select the Availability Zones you intend to use.
* Security Group:
   - Create a new Security Group.
   - Allows HTTP/HTTPS and a custom TCP rule with port range 9876
* Listeners and Routing:
   - Go to 'Listeners and routing'.
   - Select the target group you previously created.
* Click Create Load Balancer.

### 10. Create an Auto Scaling Group (ASG)

* Navigate to Auto Scaling Groups and Click the “Create Auto Scaling group” button to start the setup process.
* Name your ASG and select the launch template you previously created.
* Select the same VPC and Availability Zones (AZs) that your EC2 instance is in.
* Attach to Load Balancer:
  - Find the section to attach the ASG to a load balancer and select your existing Load Balancer from the list.
* Configure Health Checks:
  - Enable ELB (Elastic Load Balancing) health checks by selecting the appropriate option.
  - Set the health check grace period to 120 seconds. This gives instances time to start and begin serving requests before health checking starts.
* Set Group Size:
  - Define the desired, minimum, and maximum number of instances for the group:
```sh
Desired: 2
Minimum: 1
Maximum: 3
```
  - skipping scaling policies
* Add Notifications:
  - create a new SNS topic where you can enter a topic name.
  - Add your email address as the recipient for notifications.
  - Choose event types for which you wish to receive notifications, such as instance launch or termination events.
  - Confirm the subscription to the SNS topic via the email AWS sends to your email address.
  - Review all setting and create ASG.

### 11. Creating a dynamic scaling policy for your Auto Scaling Group (ASG)

* Navigate to ASG: Under "Auto Scaling," select "Auto Scaling Groups."
* Select Your ASG and Go to the “Automatic Scaling” tab, Click on “Dynamic scaling policies.”
* Select “Simple Scaling” as the policy type.
* Name Your Policy
* Set Up CloudWatch Alarm:
  - Click to create a new CloudWatch alarm.
  - Navigate to EC2 metrics, then to “By Auto Scaling Group,” and select your ASG.
  - Choose the “CPUUtilization” metric.
* Configure Metric Condition: Set the metric condition to trigger at the threshold you desire (for example, 50% CPU utilization).
* Notification:
You can set up a notification topic at this point by providing your email and setting a name for the alarm. AWS will send a notification when the alarm state changes.
* Create Alarm and Return:
  - Complete the CloudWatch alarm setup and click “Create Alarm.”
  - Return to the Dynamic Scaling Policy setup in the ASG.
* Refresh the CloudWatch Alarm list, and select the new alarm you just created. 
* Set the scaling action, for instance, to add 1 capacity unit (instance) when the alarm triggers.
* Set the cooldown period (e.g., 60 seconds). The cooldown period ensures that the ASG doesn't launch or terminate additional instances before the previous ones have had time to impact the metric.
* Click Create to finalize the dynamic scaling policy.

Once you have created the policy, the ASG will automatically adjust its size in response to the CloudWatch alarm. When the CPUUtilization goes above 50%, it will trigger the policy to add capacity. The cooldown period helps to prevent the ASG from adding too much capacity too quickly, allowing the newly launched instances to start handling traffic and reduce the overall CPU utilization.

### 12. Run the instance stress test Python script
* Connect to your EC2 instance that is part of the ASG via SSH.
```sh
ssh -i /path/to/your-key.pem ec2-user@your-ec2-ip-address
```
* Once connected, type the following command to create and edit a new file named stress.py:
```sh
nano stress.py
```
* `nano` will open with a new, blank file. Copy the Python and paste it into the nano editor window:
```sh
#!/usr/bin/env python
"""
Produces load on all available CPU cores
"""

from multiprocessing import Pool
from multiprocessing import cpu_count

def f(x):
    while True:
        x*x

if __name__ == '__main__':
    processes = cpu_count()
    print('utilizing %d cores\n' % processes)
    pool = Pool(processes)
    pool.map(f, range(processes))
```
* After pasting the content, press `CTRL + O` to write the changes to the file, press `Enter` to confirm the file name, and then `CTRL + X `to exit nano.
* You now have a file named `stress.py`. Before you can execute this script, you may need to make it executable and ensure that you have the required version of Python installed. To make the script executable, run:
```bash
chmod +x stress.py
```
* To execute the script, use:
```bash
./stress.py
```

## Appendix

![Architecture Diagram](main/../Untitled Diagram v3.png)


