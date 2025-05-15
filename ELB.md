What is AWS ELB?
Elastic Load Balancer (ELB) is a fully managed load balancer that distributes incoming traffic across multiple targets (EC2s, containers, IPs) in one or more Availability Zones.

➡️ It helps you:

Improve availability

Achieve fault tolerance

Enable scalability

Add security (via HTTPS termination, WAF)

⚙️ Types of Load Balancers
Type	Use Case	Layer	Target Support
ALB (Application)	HTTP/HTTPS traffic (Web apps, APIs)	Layer 7	EC2, ECS, IP, Lambda
NLB (Network)	High-performance TCP/UDP	Layer 4	EC2, IP
CLB (Classic)	Old generation (use ALB/NLB now)	Layer 4 + 7	EC2
GWLB (Gateway)	Deploy transparent network appliances	Layer 3	IP-based appliances

💡 Real-Time Use Cases
Scenario	Solution
Host multiple microservices on one domain	ALB with path-based routing
Gaming servers needing ultra-low latency	NLB
Legacy web apps	CLB or ALB
Deploy WAF & HTTPS on entry point	ALB with SSL Termination
Route traffic to healthy servers only	Health checks on targets
Autoscaling backend	ELB auto-distributes new instances

🧰 Key Features
Health Checks: Automatically route only to healthy targets

Sticky Sessions: Keep a user session on the same server

SSL Termination: Offload HTTPS at the ELB level

Cross-Zone Load Balancing: Balance across AZs

Path/Host Routing (ALB): /api to one group, /admin to another

Target Groups: Define sets of instances or services

🔒 ELB + Security
Use Security Groups to control access

Integrate with WAF (ALB only)

Terminate SSL on ELB and forward HTTP to backend

Use Access Logs for monitoring

🔧 ELB Configuration via AWS Console or CLI
💻 CLI to create an ALB:
bash
Copy
Edit
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-abc subnet-def \
  --security-groups sg-123456

aws elbv2 create-target-group \
  --name my-tg \
  --protocol HTTP --port 80 \
  --vpc-id vpc-xxxxxxx

aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --targets Id=i-0123 Id=i-0456

aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:...
🧪 Interview Questions and Answers
🔹 Q1: What's the difference between ALB, NLB, and CLB?
Answer:

ALB: Layer 7, HTTP/HTTPS, routing, WAF support, Lambda targets

NLB: Layer 4, TCP/UDP, very fast, static IP

CLB: Deprecated, basic Layer 4/7

🔹 Q2: How does ALB support microservices?
Answer:
ALB supports:

Host-based routing: api.example.com → one target group

Path-based routing: /login, /admin → different target groups
Great for containerized apps on ECS or Kubernetes.

🔹 Q3: What happens if a target becomes unhealthy?
Answer:
ELB removes unhealthy targets based on health checks and stops sending traffic until it becomes healthy again.

🔹 Q4: How do you achieve session stickiness?
Answer:
Enable sticky sessions on the listener → ELB uses a cookie to route the user to the same instance.

🔹 Q5: How can you terminate SSL at the ALB?
Answer:

Upload SSL certificate to ACM.

Configure HTTPS listener on ALB.

ALB decrypts the traffic, forwards plain HTTP to backend.

🔹 Q6: What is a Target Group?
Answer:
It’s a group of targets (EC2, Lambda, IPs) that ELB forwards traffic to. Each listener rule maps to a target group.

🔹 Q7: What’s the role of health checks in ELB?
Answer:
ELB continuously checks target health using HTTP/TCP and routes traffic only to healthy targets.

🔹 Q8: How does cross-zone load balancing help?
Answer:
Distributes traffic evenly across instances in all AZs, even if some AZs have fewer instances.

🔥 Real-Time Scenario
🧱 Architecture:
App: Node.js + React app

Backend EC2s in Auto Scaling group (2 AZs)

ALB in front with:

/api/* → backend service

/admin/* → admin service

💥 Result:
Scalable, highly available

SSL termination on ALB

Sticky session enabled for admin

🧼 Troubleshooting Tips
Issue	Check
504 Gateway Timeout	Backend not responding or wrong health check path
No traffic to EC2	Instance not in target group or marked unhealthy
HTTPS not working	ACM cert attached? Listener on port 443?
Sticky sessions fail	Cookies disabled in app or misconfigured
