Let’s dive deep into AWS CLI, including MFA, CloudShell, and Security Tools like Access Analyzer, Credential Reports, and more — with simple explanations, real-time use cases, and interview Q&A.

✅ 1. Overview: AWS CLI (Command Line Interface)
AWS CLI is a tool that lets you interact with AWS services via terminal/command line.
You can create, configure, and manage AWS resources without using the web console.

📦 Basic CLI Structure
bash
Copy
Edit
aws <service> <operation> [parameters]
Example:

bash
Copy
Edit
aws s3 ls             # List S3 buckets
aws ec2 describe-instances
aws iam list-users
🔐 Configuration
bash
Copy
Edit
aws configure
You’ll provide:

Access Key ID

Secret Access Key

Region (e.g., us-east-1)

Output format (json, text, table)

🔒 2. MFA (Multi-Factor Authentication) with AWS CLI
✅ Use Case:
You want to enforce extra security on CLI users by requiring MFA.

🔧 Step-by-Step: Using MFA in AWS CLI
Enable MFA on the IAM user (from Console).

Create a Session Token using sts get-session-token:

bash
Copy
Edit
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/your-user-name \
  --token-code 123456
serial-number: Your MFA device ARN

token-code: 6-digit code from MFA app

This command gives you:

AccessKeyId

SecretAccessKey

SessionToken

Export them as environment variables or use a named profile:

bash
Copy
Edit
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
✅ Now, all CLI commands will work with MFA-protected credentials.

🧰 3. AWS CloudShell
📘 What is CloudShell?
Browser-based shell provided by AWS.

Comes pre-installed with AWS CLI, Python, Git, Docker, etc.

No setup needed — just click and use.

Automatically authenticated to the currently signed-in user.

🔍 Real-Time Uses
Use Case	CloudShell Role
Testing CLI quickly	No setup needed
Running scripts for resource cleanup	Easily accessible
MFA-free CLI session (if logged in via SSO)	Convenient for temporary tasks

🔐 4. Security Tools in AWS CLI
🔸 1. IAM Credential Report
Shows password age, access keys, MFA status for all users.

bash
Copy
Edit
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | base64 --decode
➡️ Use it to find:

Users without MFA

Unused credentials

Old keys

🔸 2. IAM Access Analyzer
Identifies resources that are accessible publicly or cross-account.

bash
Copy
Edit
aws accessanalyzer list-analyzers
aws accessanalyzer list-findings
🔸 3. Security Token Service (STS)
Use to assume roles, enable temporary sessions, or test cross-account access.

bash
Copy
Edit
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT_ID:role/RoleName \
  --role-session-name session1
🔸 4. CloudTrail Insights via CLI
CloudTrail logs all API actions:

bash
Copy
Edit
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=chintu
🎯 5. Common Interview Questions + Answers
🔹 Q1: What is AWS CLI and why is it useful?
Answer:
AWS CLI allows command-line access to AWS services, enabling scripting, automation, and fast resource management without needing the AWS Console.

🔹 Q2: How do you use MFA with AWS CLI?
Answer:
You use aws sts get-session-token with the MFA device ARN and token code to generate temporary credentials. These credentials are then exported to run CLI commands securely.

🔹 Q3: What is CloudShell and how is it different from CLI on local?
Answer:
CloudShell is a browser-based terminal provided by AWS. It already includes CLI tools and is pre-authenticated — great for quick tasks without local setup.

🔹 Q4: How do you list all EC2 instances using CLI?
bash
Copy
Edit
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name]' --output table
🔹 Q5: How do you identify IAM users without MFA via CLI?
Answer:
Use the Credential Report:

bash
Copy
Edit
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | base64 --decode | grep "false"
🔥 6. Advanced / Real-Time Scenario Questions
🔸 Q6: You need to rotate IAM keys for a user using CLI. What do you do?
bash
Copy
Edit
aws iam create-access-key --user-name chintu
# Use and validate the new keys
aws iam delete-access-key --access-key-id OLD_KEY_ID --user-name chintu
🔸 Q7: You’re troubleshooting a user who cannot access a service. What CLI tools help?
Use aws iam simulate-principal-policy

Review logs via aws cloudtrail lookup-events

Check effective permissions using Access Analyzer

🔸 Q8: How do you automate EC2 snapshots via CLI?
bash
Copy
Edit
aws ec2 create-snapshot --volume-id vol-xxxx --description "Nightly backup"
🔸 Q9: How do you assume a role in another account and use it to list S3 buckets?
bash
Copy
Edit
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/CrossAccountRole \
  --role-session-name testSession

# Set credentials from output, then:
aws s3 ls
🧾 Summary Cheatsheet
Tool	Purpose
aws configure	Set up CLI credentials
sts get-session-token	MFA-protected access
CloudShell	Web-based terminal
Credential Report	Audit IAM credentials
Access Analyzer	Detect public/cross-account access
CloudTrail	View API calls
STS	Assume roles (cross-account, temporary access)
