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
   - Subnet name: twoge-pub-2a
   - Availability Zone: us-west-2a
   - IPv4 VPC CIDR block: 10.0.0.0/25
* Subnet2
   - Choose VPC ID that you just created.
   - Subnet name: twoge-pub-2b
   - Availability Zone: us-west-2b
   - IPv4 VPC CIDR block: 10.0.0.128/25
NOTE: [Visual subnet calculator](https://www.davidc.net/sites/default/subnets/subnets.html)
* Create Internat Gateway and Attach to VPC
   - Name: twoge-igw-prd
   - Click "Create internet gateway"
   - Attach to VPC- xin-twoge-vpc-uw2
   - Click "Attach internat gateway"
* Create route table
   - Name: twogee-rt-prd
   - VPC is xin-twoge-vpc-uw2
   - Go to Routes tab and Click Edit routes
   - Add route: 
   - Destination: 0.0.0.0/0 
   - Target: Internet Gateway- twoge-igw-prd
* Subnet Associate with route table
   - Go to Subnet associations
   - Add twoge-pub-2a and twoge-pub-2a and save it

### 2. Create an IAM role for S3 access.
   - Go to the IAM dashboard.
   - Create Role: Click Create role > AWS service > EC2
   - Permission: Select AmazonS3FullAccess and attach it.
   - Give a name(TwogeS3AccessRole-xin) and description and create role

### 3. Host the static files on an S3 bucket.
   - Create Bucket
   - Name: xin-twoge-s3-uw2
   - Region: us-west-2
   - Uncheck-Block all public access
   - Keep the default setting
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
  - Upload the files or folder
  - Test the list of S3 bucket 
```sh
aws s3 ls s3://xin-twoge-s3-uw2
```

### 






## Appendix

![Architecture Diagram](main/../Untitled%20Diagram.png)


