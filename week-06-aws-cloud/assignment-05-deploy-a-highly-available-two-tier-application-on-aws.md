# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS (VPC + ALB + ASG + Multi-AZ RDS)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will design and deploy a highly available two-tier web application on AWS: highly available networking across two Availability Zones, an Application Load Balancer, an Auto Scaling Group for the web tier, and a private Multi-AZ RDS database. You must prove high availability with real failure tests.

---

# Task 1 — Create HA Networking (VPC + 4 Subnets + IGW + NAT + Route Tables)

## Goal

Build a VPC (10.0.0.0/16) with two public and two private subnets across two Availability Zones, an Internet Gateway, a NAT Gateway, and the matching public/private route tables.

### Evidence

#### Screenshot 1 — VPC details showing CIDR 10.0.0.0/16

![alt text](screenshots/ha-vpc.jpg)

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![alt text](screenshots/four-sunets.jpg)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

 ![alt text](screenshots/public-gw-subnets.jpg)

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![alt text](screenshots/private-connections.jpg)

---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![alt text](screenshots/NAT-status.jpg)

---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![alt text](screenshots/alb-sg-inbound.jpg)

---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![alt text](screenshots/Ec2-sg-inbound.jpg)

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![alt text](screenshots/RDS-sg-inbound.jpg)

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![ ](screenshots/single-AZ.jpg)

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![alt text](screenshots/endpoint-db.jpg)

---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![alt text](screenshots/web-template.jpg)
![alt text](screenshots/web-template-userdata.jpg)

---

#### Screenshot 12 — A running instance created from the template showing that the application responds on port 80 through a local test or browser using its public IP

![alt text](screenshots/wordpress-test.jpg)
---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![alt text](screenshots/ha-alb-2availzones.jpg)

---

#### Screenshot 14 — Target group showing at least one healthy target

![alt text](screenshots/tg-gp-health-checks.jpg)

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![alt text](screenshots/ha-asg-capacity.jpg)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![alt text](screenshots/instances-2AZ.jpg)

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![alt text](<screenshots/app-ALB-DNS name.jpg>)

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![alt text](screenshots/wordpress-install.jpg)
![alt text](screenshots/new-post.jpg)

---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB.

Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance; timestamps are helpful

![alt text](screenshots/SS-19.jpg)

---

#### Screenshot 20 — Target group showing healthy targets after replacement

![alt text](screenshots/targets.jpg)

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![alt text](screenshots/standby.jpg)

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![alt text](screenshots/still-working.jpg)

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram, which may be hand-drawn, or an AWS console overview showing the components

![alt text](screenshots/Architecture.jpg)

---

### Notes

Summarize the VPC and subnets across the two Availability Zones.

The application was deployed inside ha-vpc (10.0.0.0/16) in the AWS London region. The network was designed across two Availability Zones, eu-west-2a and eu-west-2b, with two public and two private subnets. The public subnets were connected to the Internet Gateway through the public route table, while the private subnets used a NAT Gateway for outbound internet access. This provided network separation between the public web tier and the private database tier.

Summarize the ALB and Auto Scaling Group setup.

An internet-facing Application Load Balancer (ha-alb) was deployed across both public subnets and configured with an HTTP listener on port 80. Traffic was forwarded to the ha-web-tg target group containing EC2 web instances. The Auto Scaling Group (ha-asg) used the HA-WEB-Launch-Template to maintain 2 desired instances, minimum 2 and maximum 4, distributed across the two Availability Zones. ELB health checks were enabled so unhealthy instances could be detected and automatically replaced.

Summarize the private Multi-AZ RDS setup.

Amazon RDS for MySQL was deployed as ha-db with the application database appdb. The database was placed in the private DB subnet group with Public access disabled and protected by ha-db-sg, which permits MySQL traffic on port 3306 only from the web-tier security group. The original assignment required Multi-AZ RDS; however, Multi-AZ was unavailable under the AWS Free plan. So, a Single-AZ RDS instance was used while maintaining the required private-network and least-privilege security configuration.

Summarize the results of both high-availability tests.

Test A — Instance failure: One Auto Scaling EC2 instance was deliberately terminated. The ALB continued routing traffic to the remaining healthy instance, while the Auto Scaling Group detected the failure and automatically launched a replacement. The target group eventually returned to two healthy instances, and the website remained available.

Test B — Availability Zone/web-instance impact: One instance was placed into Standby, temporarily removing it from service. The ALB continued routing traffic to the healthy instance in the other Availability Zone, and the WordPress site remained accessible through the ALB DNS endpoint. This demonstrated high availability of the web tier across two Availability Zones.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/shanmuki-reddy_aws-devops-cloudcomputing-ugcPost-7494488254562488320-BWq8/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAE0LbgwBcO3gizrVfuqLPvGD60OHg7LFHRw
---

#### Screenshot of LinkedIn post

![alt text](screenshots/LinkedIn-SS.jpg)

---

# Submission Instructions

- Add all required screenshots in your submission
- Do not expose passwords, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: VPC, four subnets, IGW, NAT Gateway, and route tables created (Screenshots 1–5)
- [ ] Task 2: Least-privilege ALB, EC2, and RDS security groups created (Screenshots 6–8)
- [ ] Task 3: Private Multi-AZ RDS created (Screenshots 9–10)
- [ ] Task 4: Self-configuring Launch Template created and tested (Screenshots 11–12)
- [ ] Task 5: ALB created across both public subnets (Screenshots 13–14)
- [ ] Task 6: Auto Scaling Group running two instances across two AZs (Screenshots 15–16)
- [ ] Task 7: Application verified through the ALB with a database read and write (Screenshots 17–18)
- [ ] Task 8: Both high-availability tests completed (Screenshots 19–22)
- [ ] Task 9: Architecture and test-results summary completed (Screenshot 23 & Notes)
- [ ] LinkedIn post published and URL submitted
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*