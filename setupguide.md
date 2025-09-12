Step 1: Create VPC and Network Infrastructure 1.1 Create VPC

Console Steps:

Go to VPC Dashboard → Create VPC Name: moodle-vpc IPv4 CIDR: 10.0.0.0/16 IPv6 CIDR: No IPv6 Tenancy: Default
<img width="1912" height="792" alt="image" src="https://github.com/user-attachments/assets/cead034d-20d8-4335-89b8-0a4e16324570" />


1.2 Create Subnets (4 subnets total)

Public Subnets:

Public Subnet AZ-A: 10.0.1.0/24 (us-east-1a) Public Subnet AZ-B: 10.0.2.0/24 (us-east-1b)

Private Subnets:

Private Subnet AZ-A: 10.0.11.0/24 (us-east-1a) Private Subnet AZ-B: 10.0.12.0/24 (us-east-1b)

<img width="1910" height="790" alt="image" src="https://github.com/user-attachments/assets/cacdd626-51b7-46ee-9a2b-d9030d641f8b" />

1.3 Create Internet Gateway

Create Internet Gateway → Name: moodle-igw Attach to moodle-vpc
<img width="1902" height="790" alt="image" src="https://github.com/user-attachments/assets/f457be56-d4f9-4707-bb9f-36f6002e34e4" />

1.4 Create NAT Gateways

Create NAT Gateway in each public subnet NAT-AZ-A in Public Subnet AZ-A NAT-AZ-B in Public Subnet AZ-B Allocate Elastic IPs for each
<img width="1906" height="756" alt="image" src="https://github.com/user-attachments/assets/6b7f458f-ecd9-43c8-a464-dda07819dde2" />


1.5 Configure Route Tables

Public Route Table:

Route: 0.0.0.0/0 → Internet Gateway Associate with public subnets

Private Route Tables (create 2):

Private RT AZ-A: 0.0.0.0/0 → NAT Gateway AZ-A Private RT AZ-B: 0.0.0.0/0 → NAT Gateway AZ-B

<img width="1905" height="723" alt="image" src="https://github.com/user-attachments/assets/ef6a68da-e4cf-4e30-a3ea-e3dbbdecbbdd" />

Step 2: Create Security Groups 2.1 ALB Security Group

Name: moodle-alb-sg

Inbound Rules:

HTTP (80) - 0.0.0.0/0

HTTPS (443) - 0.0.0.0/0

2.2 EC2 Security Group

Name: moodle-ec2-sg

Inbound Rules: HTTP (80) - ALB Security Group

SSH (22) - Your IP (for management)

2.3 Database Security Group

Name: moodle-db-sg Inbound Rules:

MySQL (3306) - EC2 Security Group

2.4 EFS Security Group

Name: moodle-efs-sg

Inbound Rules:

NFS (2049) - EC2 Security Group
<img width="1900" height="807" alt="image" src="https://github.com/user-attachments/assets/05e8c587-67b6-4bb7-b1f2-3eec3d0b35dd" />
Step 3: MySql Databas:-
3.1 Create DB Subnet Group

Go to RDS → Subnet Groups → Create

Name: HA-db-subnet-group

VPC: Select your VPC

Add both private subnets

3.2 Create MySql

RDS → Create Database

Engine: MySql

Version: Latest

Templates: Production

Cluster identifier:HA-MySql

Master username: admin

Master password: [secure-password]

Instance class: db.t3.medium (start small, can scale)

Multi-AZ: Yes

VPC: Your VPC

Subnet group: HA-db-subnet-group

Security group: HA-db-sg 
<img width="1900" height="722" alt="image" src="https://github.com/user-attachments/assets/05f690bc-fd62-4ff9-af9a-14388eddf8ad" />
<img width="1902" height="740" alt="image" src="https://github.com/user-attachments/assets/16fe6cb1-3a5f-4a75-8cef-c23dbd6ec93d" />
Step 4: Create EFS File System
4.1 Create EFS

EFS → Create File System

Name: moodle-efs

VPC: Your VPC

Availability and Durability: Regional

Performance mode: General Purpose

Throughput mode: Provisioned

4.2 Create Mount Targets

Create mount targets in both private subnets

Security group: moodle-efs-sg
<img width="1912" height="752" alt="image" src="https://github.com/user-attachments/assets/550c065d-5867-486d-a67c-15c6b4389407" />
Step 5: Create Launch Template and Auto Scaling
5.1 Create Launch Template

bash# Name: moodle-launch-template

AMI: Amazon Linux 2023

Instance type: t3.medium

Key pair: Your key pair

Security group: moodle-ec2-sg

User Data Script:

bash#!/bin/bash

yum install httpd -y

systemctl restart httpd --now

sudo yum install -y httpd php php-mysqli mariadb

sudo yum install -y php php-cli php-mysqlnd php-pdo php-common php-gd php-xml

create index.html,db.php (Database Connection),register.php (User Registration),login.php (User Login)
<img width="1907" height="777" alt="image" src="https://github.com/user-attachments/assets/83e978ef-3db6-4da5-afe5-786037492b81" />

5.2 Create Auto Scaling Group
EC2 → Auto Scaling Groups → Create

Name: moodle-asg

Launch template: moodle-launch-template

VPC: Your VPC

Subnets: Both private subnets

Desired: 2, Min: 2, Max: 6

Health check: ELB

Health check grace period: 300 seconds

<img width="1910" height="761" alt="image" src="https://github.com/user-attachments/assets/34ba24c2-9b25-441d-bc73-acd2af4b05a7" />

Step 6: Create Application Load Balancer
6.1 Create ALB

EC2 → Load Balancers → Create ALB

Name: moodle-alb

Scheme: Internet-facing

VPC: Your VPC

Subnets: Both public subnets

Security group: moodle-alb-sg
<img width="1911" height="796" alt="image" src="https://github.com/user-attachments/assets/fdfada4d-516b-4704-b6c8-6a777c76bf86" />
6.2 Create Target Group

Name: moodle-targets

Protocol: HTTP

Port: 80

VPC: Your VPC

Health check path: /moodle/ 
<img width="1915" height="757" alt="image" src="https://github.com/user-attachments/assets/d532ea2c-f7aa-4c5a-bada-c8290f9430e8" />
8.2 Route 53 (if you have a domain)

Route 53 → Hosted zones

Create A record pointing to CloudFront distribution
<img width="1907" height="796" alt="image" src="https://github.com/user-attachments/assets/3d72b544-56e8-4b8d-9b15-1551bbc25c0b" />
<img width="1897" height="856" alt="image" src="https://github.com/user-attachments/assets/76fdecc2-3c14-4daf-8b1b-4b15ac41c5be" />
<img width="1908" height="633" alt="image" src="https://github.com/user-attachments/assets/8d44c40e-e9fe-4460-a2ad-8ac9c0b9404a" />
<img width="1910" height="954" alt="image" src="https://github.com/user-attachments/assets/e54f4aa8-8eec-4a1d-95ce-7be81f942c87" />











