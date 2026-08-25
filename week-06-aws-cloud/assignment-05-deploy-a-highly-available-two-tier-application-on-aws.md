# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS

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

![Screenshot 0](screenshots/week06-ass5-00.png)

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![Screenshot 1](screenshots/week06-ass5-01.png)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

![Screenshot 2](screenshots/week06-ass5-02.png)
![Screenshot 11](screenshots/week06-ass5-11.png)

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![Screenshot 12](screenshots/week06-ass5-12.png)
![Screenshot 13](screenshots/week06-ass5-13.png)

---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![Screenshot 7](screenshots/week06-ass5-07.png)

---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![Screenshot 8](screenshots/week06-ass5-08.png)

---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![Screenshot 9](screenshots/week06-ass5-09.png)

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![Screenshot 10](screenshots/week06-ass5-10.png)

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![Screenshot 14](screenshots/week06-ass5-14.png)
![Screenshot 15](screenshots/week06-ass5-15.png)

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![Screenshot 16](screenshots/week06-ass5-16.png)

---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![Screenshot 17](screenshots/week06-ass5-000.png)

---

#### Screenshot 12 — A running instance created from the template showing the application responds on port 80

![Screenshot 18](screenshots/week06-ass5-001.png)

---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![Screenshot 19](screenshots/week06-ass5-002.png)

---

#### Screenshot 14 — Target group showing at least one healthy target

![Screenshot 20](screenshots/week06-ass5-003.png)

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![Screenshot 21](screenshots/week06-ass5-004.png)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![Screenshot 22](screenshots/week06-ass5-005.png)

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![Screenshot 23](screenshots/week06-ass5-006.png)

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![Screenshot 24](screenshots/week06-ass5-007.png)

---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB. Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance

![Screenshot 25](screenshots/week06-ass5-008.png)

---

#### Screenshot 20 — Target group showing healthy targets after replacement

![Screenshot 26](screenshots/week06-ass5-009.png)

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![Screenshot 27](screenshots/week06-ass5-010.png)

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![Screenshot 28](screenshots/week06-ass5-011.png)

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram (hand-drawn is fine), or an AWS console overview showing the components

![Screenshot 29](screenshots/week06-ass5-012.png)

---

### Notes

Write a short summary covering the network, ALB/ASG setup, RDS setup, and the results of Test A and Test B.

## Architecture Summary

The application was deployed in the `10.0.0.0/16` VPC across two Availability Zones in the `eu-west-3` Region. The web tier uses two public subnets, `10.0.1.0/24` in `eu-west-3a` and `10.0.2.0/24` in `eu-west-3b`, while the database tier uses two private subnets, `10.0.11.0/24` and `10.0.12.0/24`.

An Internet Gateway provides internet connectivity to the public subnets, while a NAT Gateway provides outbound connectivity for the private network. The Application Load Balancer is internet-facing and spans both public subnets, forwarding HTTP traffic on port 80 to the `ha-web-tg` target group.

The web tier is managed by an Auto Scaling Group with a desired capacity of 2, minimum capacity of 2, and maximum capacity of 4. The EC2 instances are distributed across the two Availability Zones to provide redundancy. The database tier uses a private Multi-AZ RDS deployment and is not publicly accessible.

Security Groups enforce the intended traffic flow: Internet → ALB → EC2 → RDS. The application was verified through the ALB DNS endpoint and successfully performed database read and write operations.

## High-Availability Test Results

### Test A — Instance Failure

One web-tier EC2 instance was intentionally terminated. The Auto Scaling Group detected the reduced capacity and automatically launched a replacement instance. The Application Load Balancer continued serving the application through the remaining healthy instance, and the replacement instance was subsequently registered with the target group and became healthy.

### Test B — Availability Zone Impact

One web-tier instance in an Availability Zone was temporarily removed/stopped/placed into the selected failure state. The remaining instance in the other Availability Zone continued serving traffic through the Application Load Balancer. The ALB DNS endpoint remained accessible during the test, demonstrating that the application could continue operating despite the simulated Availability Zone impact.

## Conclusion

The deployment demonstrated a highly available two-tier AWS architecture using Multi-AZ networking, an internet-facing Application Load Balancer, an Auto Scaling web tier, and a private Multi-AZ RDS database. Both required failure tests demonstrated that the web application could remain available and recover automatically from web-tier failures.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`https://www.linkedin.com/posts/ikechukwu-emmanuel_architecture-summary-the-application-was-activity-7498084070409654272-DItt?utm_source=social_share_send&utm_medium=member_desktop_web&rcm=ACoAAGENBPUBsLYqmgeLRkF6HTid7rCysjW2i7w`

---

#### Screenshot — Published LinkedIn post

![Screenshot LinkedIn](screenshots/week06-ass5-013.png)

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
