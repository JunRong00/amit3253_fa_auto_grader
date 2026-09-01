# AMIT3253 Cloud Computing — Auto Grader

An automated grading script for the AMIT3253 Cloud Computing assignment. It connects to a student's AWS account via `boto3`, inspects the live infrastructure they provisioned, and produces a scored report (out of 100) across four core tasks: EC2 web server deployment, Auto Scaling, Load Balancing, and RDS.

## What It Checks

The grader awards points based on real-time inspection of AWS resources, matched to the student by name (flexible naming is supported — e.g. `asg-john`, `john-asg`, `johnasg` are all recognized).

| Task | Weight | Checks |
|------|--------|--------|
| **1. EC2 Web Server** | 25 pts | Instance running, correct instance type (`t3.small`), security group rules (ports 22 & 80), site returns HTTP 200, page displays the student's name |
| **2. Launch Template & ASG** | 25 pts | Launch template exists with user data, Auto Scaling Group is linked to it, scaling config matches (min 1 / max 3 / desired 1), instances are running via the ASG |
| **3. Load Balancer** | 25 pts | Internet-facing ALB exists, HTTP:80 listener configured, target group exists, targets are registered and healthy, site is reachable via the ALB DNS name and shows the student's name |
| **4. RDS Database** | 25 pts | Identifier matches naming convention (`rds-<name>` or `<name>-rds`), engine is MySQL on `db.t3.micro`, storage is 10GB `gp2` with DB name `studentdb` and username `admin`, instance is available and not publicly accessible, security group allows MySQL (3306) from `10.0.0.0/16` |

Several checks award **partial credit** (e.g. using a close-but-incorrect instance type, or opening only one of the two required ports).

## Requirements

- Python 3.6+
- [`boto3`](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- Valid AWS credentials configured (via `aws configure`, environment variables, or an IAM role) with read access to EC2, Auto Scaling, ELBv2, and RDS in the target region

Install the dependency:

```bash
pip install boto3
```

## Usage

Run the grader from a machine or shell with AWS credentials configured for the student's account:

```bash
python amit3253_grader_202605
```

You will be prompted to enter the student's full name, which is used to locate their tagged resources:

```
Enter Student Full Name: Jane Doe
```

The script then scans the configured AWS region and prints a detailed, task-by-task breakdown, followed by a final score.

### Sample Output

```
============================================================
 Task 1: EC2 Web Server Deployment (25%)
============================================================
[✓] PASS (+5/5): EC2 Instance Launched & Running
[!] PARTIAL (+2/5): Instance Type (t3.small)
    -> Note: Wrong type used: t3.micro (Awarded 2/5)
[✓] PASS (+5/5): Security Group: Ports 22 & 80
...
============================================================
 FINAL RESULT
============================================================
TOTAL SCORE: 78 / 100
```

## Notes

- The script performs read-only AWS API calls; it does not create, modify, or delete any resources.
- HTTP checks (`urllib`) use a short timeout and an unverified SSL context, so unreachable or slow-loading sites will correctly register as failures rather than hanging the grader.
- If a task's AWS API calls raise an exception (e.g. missing permissions), that task's error is printed and grading continues with the remaining tasks.
- Name matching is case-insensitive and ignores spaces, hyphens, and underscores, so minor formatting differences in resource names or tags won't cause false negatives.

## Disclaimer

This tool is intended for instructor use in grading student-submitted AWS infrastructure for the AMIT3253 course. Ensure you have appropriate authorization before running it against any AWS account.
