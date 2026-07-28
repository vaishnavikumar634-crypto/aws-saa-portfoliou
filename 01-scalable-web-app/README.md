# Project 1: Scalable Web Application with ALB and Auto Scaling

**Architecture type:** EC2-based, multi-AZ, highly available web application

## Table of Contents
- [Solution Overview](#solution-overview)
- [Architecture Diagram](#architecture-diagram)
- [Key AWS Services](#key-aws-services)
- [How It Works](#how-it-works)
- [Deploying This Solution](#deploying-this-solution)
  - [Prerequisites](#prerequisites)
  - [1. Build the network](#1-build-the-network)
  - [2. Deploy compute and scaling](#2-deploy-compute-and-scaling)
  - [3. Deploy the database](#3-deploy-the-database)
  - [4. Add edge and DNS](#4-add-edge-and-dns)
  - [5. Add monitoring](#5-add-monitoring)
- [Learning Outcomes](#learning-outcomes)
- [Cost Considerations](#cost-considerations)
- [Possible Enhancements](#possible-enhancements)
- [License](#license)

## Solution Overview
This project deploys a production-grade web application on AWS using EC2 instances inside a properly architected VPC, with public and private subnets spread across two Availability Zones. High availability and scalability come from an Application Load Balancer (ALB) and an Auto Scaling Group (ASG), while CloudFront caches static assets at the edge. The database tier runs on Multi-AZ RDS for automated failover, and all compute lives in private subnets with no direct internet exposure.

## Architecture Diagram
![Architecture Diagram](architecture-diagram.svg)
*Figure 1: Multi-AZ VPC with public ALB tier, private application tier (EC2 in an Auto Scaling Group), and private data tier (RDS Multi-AZ), fronted by Route 53 and CloudFront.*

## Key AWS Services
| Service | Role |
|---|---|
| VPC | Public & private subnets across 2 AZs, NAT Gateway, Security Groups, NACLs |
| EC2 + Auto Scaling Group | Launch Template, target-tracking scaling policy |
| ALB + WAF | Layer 7 routing, WAF managed rules for OWASP Top 10 |
| CloudFront | Caches static assets, reduces latency globally |
| RDS Multi-AZ | MySQL/PostgreSQL with automated failover |
| Route 53 | Alias record to ALB, DNS health checks |
| Systems Manager | Session Manager for bastion-free, SSH-key-free instance access |
| CloudWatch + SNS | Dashboards, alarms, and operational notifications |

## How It Works
1. A user request hits Route 53, which resolves to the CloudFront distribution (for static assets) or directly to the ALB.
2. The ALB, sitting in public subnets across two AZs, terminates the connection and routes it through WAF rules to the target group.
3. Traffic reaches EC2 instances running in private subnets, managed by an Auto Scaling Group that scales based on CPU utilization (target tracking).
4. The application tier connects to a Multi-AZ RDS instance in an isolated private subnet; if the primary AZ fails, RDS fails over automatically to the standby.
5. CloudWatch collects metrics from every tier; alarms notify the team via SNS on abnormal error rates or scaling events.

## Deploying This Solution

### Prerequisites
- An AWS account with console/CLI access
- A domain (or subdomain) if you want to attach Route 53 + ACM

### 1. Build the network
Create the VPC with 2 public + 2 private subnets across 2 AZs, an Internet Gateway, and a NAT Gateway. Configure Security Groups so only the ALB can reach EC2, and only EC2 can reach RDS.

### 2. Deploy compute and scaling
Create a Launch Template from a configured EC2 instance and attach it to an Auto Scaling Group. Create the ALB, target group, and a WAF web ACL using AWS Managed Rules.

### 3. Deploy the database
Deploy RDS in Multi-AZ mode inside the private data subnets, reachable only from the app tier's Security Group.

### 4. Add edge and DNS
Add a CloudFront distribution in front of the ALB for static content, and point a Route 53 alias record at the ALB with a health check attached.

### 5. Add monitoring
Configure CloudWatch dashboards and SNS alarm subscriptions for error rates, scaling events, and RDS failover.

## Learning Outcomes
- Design VPCs with correct subnet, route table, and NAT Gateway configurations.
- Build highly available architectures across multiple Availability Zones.
- Configure ALB listener rules and target group health checks.
- Implement Auto Scaling with target tracking and step scaling policies.
- Secure applications with WAF, Security Groups, and private subnets.
- Use Systems Manager Session Manager as a bastion-free access alternative.

## Cost Considerations
The main hourly costs are the NAT Gateway, Multi-AZ RDS instance, and running EC2 instances. Use the smallest instance sizes for demo purposes and tear down the stack (or stop RDS/EC2) when not actively demonstrating it.

## Possible Enhancements
- Add HTTPS via an ACM certificate and a 443 listener.
- Add a step scaling policy for burst traffic in addition to target tracking.
- Automate the entire stack with CloudFormation or Terraform for repeatability.

## License
This project was built for educational purposes as part of an AWS Solutions Architecture project submission.
