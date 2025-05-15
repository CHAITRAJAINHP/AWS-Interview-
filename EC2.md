Let’s deep dive into AWS EC2 (Elastic Compute Cloud) — one of the most important and commonly asked topics in AWS interviews. Below you'll find:

✅ Overview

💡 Real-time Use Cases

📦 Key Features

🔐 Security Concepts

🔧 CLI Commands

🧠 Interview Q&A (basic to advanced)

🧪 Troubleshooting tips

📘 Markdown-ready version for GitHub (optional – tell me if you want it)

✅ 1. What is EC2?
Amazon EC2 is a web service that provides resizable compute capacity in the cloud. It's like a virtual server that you can spin up and use on-demand.

💡 Real-Time Use Cases
Use Case	EC2 Role
Host web apps	Run backend servers (Node.js, Django, etc.)
CI/CD agents	Used as Jenkins workers or GitHub runners
Development/Test environments	Quick test VMs for QA
Batch processing	Transcoding videos, ML training jobs
VPN or bastion hosts	Secure internal access to private VPC resources

📦 2. Key EC2 Concepts
Term	Description
AMI	Amazon Machine Image (OS + preinstalled software)
Instance Type	Defines CPU, memory (e.g., t2.micro, m5.large)
Key Pair	Used for SSH login to EC2
Security Group	Acts as a virtual firewall
EBS	Elastic Block Store – persistent storage volume
Elastic IP	Static public IP for EC2 instance
User Data	Bootstrap shell script at instance launch
Placement Group	Logical grouping of instances (cluster/spread)
Launch Template	Predefined EC2 config for Auto Scaling or launches

🔐 3. EC2 Security Best Practices
Use Security Groups to restrict access (e.g., allow SSH only from office IP).

Use IAM roles instead of hardcoding credentials.

Enable CloudWatch Logs/Alarms for monitoring.

Use EC2 Instance Connect instead of key pairs when possible.

Never expose port 22 (SSH) publicly without restriction.

🧠 4. EC2 Interview Questions + Answers
🔹 Q1: What is the difference between an AMI and an instance?
Answer:
An AMI is a template used to launch EC2 instances. It includes OS, app server, and applications.
An instance is a running virtual server launched using an AMI.

🔹 Q2: What happens when you stop and start an EC2 instance?
Answer:

Instance stops → compute billing stops, EBS billing continues.

IP changes (unless Elastic IP used).

Ephemeral storage (instance store) data is lost.

User data scripts do not run again.

🔹 Q3: How do you automate software installation on EC2?
Answer:
Use User Data (bash script) at launch:

bash
Copy
Edit
#!/bin/bash
yum update -y
yum install httpd -y
systemctl enable httpd
systemctl start httpd
🔹 Q4: What’s the difference between Security Groups and NACLs?
Feature	Security Group	NACL
Scope	Instance-level	Subnet-level
Stateful	✅ Yes	❌ No
Rules	Allow only	Allow + Deny
Direction	Inbound + Outbound	Inbound + Outbound separately

🔹 Q5: How can you access a private EC2 instance?
Answer:
Via a Bastion Host (Jump Server):

Connect to bastion in public subnet → then SSH to private instance in private subnet.

🔹 Q6: What is EC2 Spot vs On-Demand vs Reserved?
Type	Description	Use Case
On-Demand	Pay hourly, no commitment	Short-term workloads
Reserved	1/3-year commitment, cheaper	Steady state apps
Spot	Unused EC2 at 90% discount	Flexible, fault-tolerant tasks

🔹 Q7: Can EC2 access S3 without credentials?
Yes, via IAM Role assigned to the instance:

json
Copy
Edit
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}
🔹 Q8: What is EC2 Instance Metadata?
You can query internal data from inside EC2:

bash
Copy
Edit
curl http://169.254.169.254/latest/meta-data/
Example use:

Get instance ID

Get IAM role name

Check public IP

🧪 Troubleshooting Tips
Problem	Check
Can’t SSH	Security group rules, key pair, instance status
No internet	Public IP, route table, NAT gateway
App not working	Inbound port in Security Group open?
Boot issue	Check System Logs in EC2 Console
Can’t assume role	EC2 IAM role attached? Instance Profile valid?

🔧 5. Useful AWS CLI Commands
bash
Copy
Edit
# List EC2 instances
aws ec2 describe-instances --query 'Reservations[*].Instances[*].InstanceId'

# Start/Stop EC2
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Create key pair
aws ec2 create-key-pair --key-name MyKey --query 'KeyMaterial' --output text > MyKey.pem

# Launch instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --key-name MyKey \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --user-data file://bootstrap.sh
✍️ Bonus: User Data for Apache Setup
bash
Copy
Edit
#!/bin/bash
yum update -y
yum install httpd -y
echo "<h1>Hello from EC2 $(hostname -f)</h1>" > /var/www/html/index.html
systemctl start httpd
systemctl enable httpd
