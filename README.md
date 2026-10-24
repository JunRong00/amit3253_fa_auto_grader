# AMIT3253 Cloud Computing — FA Auto Grader

Automated grading scripts for the AMIT3253 Cloud Computing for Business Final Assessment (FA). Each script connects to a student's AWS Academy Learner Lab account via `boto3`, inspects the live infrastructure they provisioned, and produces a scored report out of 100.

| Script | Assessment | Questions |
|--------|------------|-----------|
| [`amit3253_grader_202609`](amit3253_grader_202609) | **Current** — Final Assessment (December, Week 6) | VPC & Networking, EC2, Application Load Balancer, Auto Scaling Group |
| [`amit3253_grader_202605`](amit3253_grader_202605) | Previous (May 2026) | EC2 Web Server, Launch Template & ASG, Load Balancer, RDS |

## What It Checks (202609)

Resources are matched to the student by name. Matching is case-insensitive and ignores spaces, hyphens and underscores, so `vpc-JaneDoe`, `jane_doe-vpc` and `vpcjanedoe` are all recognized for the student "Jane Doe".

### Question 1 — VPC and Networking (25 marks)

| Criterion | Marks | Expected |
|-----------|-------|----------|
| 1.1 VPC Name | 4 | Contains `vpc` and the student's name (wizard `-vpc` suffix accepted) |
| 1.2 VPC CIDR Block | 3 | `10.0.0.0/16` |
| 1.3 Availability Zones | 4 | Subnets in 3+ AZs (2 marks for exactly 2) |
| 1.4 Public Subnets | 3 | `10.0.1.0/24`, `10.0.2.0/24`, `10.0.3.0/24` (1 mark each) |
| 1.5 Private Subnets | 3 | `10.0.11.0/24`, `10.0.12.0/24`, `10.0.13.0/24` (1 mark each) |
| 1.6 Internet Gateway | 2 | Created, and attached to the VPC |
| 1.7 Public Route Table | 3 | Route to an IGW, destination `0.0.0.0/0`, targeting the VPC's IGW |
| 1.8 Route Table Association | 3 | Public route table associated with each public subnet (1 mark each) |

### Question 2 — EC2 Instance (25 marks)

The standalone instance is graded; ASG-managed instances are excluded.

| Criterion | Marks | Expected |
|-----------|-------|----------|
| 2.1 Instance Name | 3 | Contains `ec2` and the student's name |
| 2.2 AMI | 3 | Amazon Linux 2023 |
| 2.3 Instance Type | 3 | `t3.large` |
| 2.4 VPC/Subnet Placement | 2 | In the Q1 VPC, in a Q1 public subnet |
| 2.5 Public IP Auto-assign | 1 | Instance has a public IP |
| 2.6 Security Group Name | 1 | `<name>-sg-ec2` |
| 2.7 Security Group Rules | 3 | Inbound HTTP, SSH and MySQL (1 mark each) |
| 2.8 Root Volume | 1 | 15 GiB `gp2` |
| 2.9 IAM Instance Profile | 4 | Profile attached (1), and it is `LabInstanceRole` (3) |
| 2.10 User Data Script | 4 | Installs httpd (1), starts/enables httpd (1), serves personalised content (2) |

The script also fetches the instance's public IP and reports whether the page loads and shows the student's name (informational, not scored).

### Question 3 — Application Load Balancer (25 marks)

| Criterion | Marks | Expected |
|-----------|-------|----------|
| 3.1 Target Group Name | 3 | Contains `tg` and the student's name |
| 3.2 Target Type | 3 | `instance` |
| 3.3 TG Protocol/Port | 2 | HTTP, port 80 |
| 3.4 TG VPC | 1 | The Q1 VPC |
| 3.5 Health Check | 2 | HTTP, path `/` |
| 3.6 Load Balancer Name | 3 | Contains `alb` and the student's name |
| 3.7 ALB Type/Scheme/IP | 2 | Application, internet-facing (1); IPv4 (1) |
| 3.8 Availability Zones | 2 | Two AZs mapped to Q1 public subnets |
| 3.9 ALB Security Group | 2 | Named `<name>-sg-alb` (1), allows inbound HTTP (1) |
| 3.10 Listener | 2 | HTTP, port 80 |
| 3.11 Default Action | 3 | Action exists (1), forwards to the Q3 target group (2) |

The ALB DNS name is also fetched as an informational live check.

### Question 4 — Auto Scaling Group (25 marks)

| Criterion | Marks | Expected |
|-----------|-------|----------|
| 4.1 Launch Template Name | 3 | Contains `lt` and the student's name |
| 4.2 LT Config Matches Q2 | 4 | Same AMI, instance type and httpd user data as Q2 (2 marks if only 2 of 3 match) |
| 4.3 ASG Name | 3 | Contains `asg` and the student's name |
| 4.4 ASG VPC | 1 | The Q1 VPC |
| 4.5 ASG Subnets | 3 | Two Q1 public subnets in distinct AZs (1 mark if only one is correct) |
| 4.6 ASG Security Group | 5 | Named correctly (1), allows port 80 (2), source is the ALB security group (2) |
| 4.7 Capacity | 3 | Min 1 / Desired 1 / Max 3 (1 mark if exactly one value is wrong) |
| 4.8 Target Group Attachment | 3 | A target group is attached (1), and it is the Q3 target group (2) |

Later questions reuse what earlier ones found (the Q1 VPC and public subnets, the Q2 AMI and user data, the Q3 target group and ALB security group), so a missing resource early on can cost marks downstream.

## Requirements

- Python 3.6+
- [`boto3`](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- AWS credentials for the student's account with read access to EC2, Auto Scaling and ELBv2. The Learner Lab's exported AWS CLI credentials are sufficient.

```bash
pip install boto3
```

## Usage

Run the grader inside the student's AWS Academy Learner Lab session, in the same region the resources were created in:

```bash
python3 amit3253_grader_202609
```

Enter the student's full name when prompted:

```
Enter Student Full Name: Jane Doe
```

The script prints a criterion-by-criterion breakdown for each question, followed by the final score.

### Sample Output

```
======================================================================
QUESTION 1: VPC AND NETWORKING (25 marks)
======================================================================
[OK] 1.1 VPC Name: 4/4  (vpc-janedoe-vpc)
[OK] 1.2 VPC CIDR Block: 3/3  (10.0.0.0/16)
[~] 1.3 Availability Zones: 2/4  (2 AZ(s) used)
...
======================================================================
FINAL SCORE SUMMARY
======================================================================
Student: Jane Doe
TOTAL SCORE: 82 / 100
Percentage: 82.0%
```

`[OK]` is full marks, `[~]` is partial credit and `[X]` is zero.

## Notes

- The auto-grader gives a **first-pass score only**. If its output disagrees with the student's submitted screenshot evidence, the marking rubric (`AMIT3253_Marking_Rubric.md`) is the reference for manual adjustment.
- The script makes read-only AWS API calls; it never creates, modifies or deletes resources.
- HTTP checks use a 10-second timeout and an unverified SSL context, so unreachable sites register as failures instead of hanging the grader.
- If a question's AWS API calls raise an exception (for example, missing permissions), the error is printed and grading continues with the remaining questions.

## Disclaimer

This tool is intended for instructor use in grading student-submitted AWS infrastructure for the AMIT3253 course. Ensure you have appropriate authorization before running it against any AWS account.
