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




Target Groups — 💡 "Who should the ELB send traffic to?"
✅ What is a Target Group?
A Target Group is a collection of targets (like EC2 instances, IP addresses, Lambda functions, or containers) that the ELB can forward requests to.

🔧 Each target group is linked to:
A protocol (HTTP, HTTPS, TCP)

A port (e.g., 80, 443)

A health check path (e.g., /health)

📦 Real Example:
You launch 3 EC2 instances to run a Node.js app.

You create a Target Group called web-app-tg and register those 3 instances in it.

Then, you attach that Target Group to an ALB Listener Rule (like for /api).

➡️ So when users access your app, the ELB sends traffic only to healthy instances in the target group.

🔥 Why Target Groups?
Flexibility: You can send /api to one group and /admin to another.

Health Checks are run per target group.

Supports blue/green deployments (switch groups without changing ELB).

2️⃣ Sticky Sessions — 🍪 "Keep me talking to the same server"
✅ What is a Sticky Session?
A sticky session (a.k.a. session affinity) ensures that once a user connects to a backend, they keep connecting to the same target for the duration of their session.

This is done using cookies.

🔧 How it works:
When a user connects:

ELB sends them to EC2 instance A

A special cookie (AWSALB) is stored in their browser

Future requests go to instance A (as long as the cookie is valid)

📦 Real Example:
You have a shopping cart web app.

Without sticky sessions: User adds items → request goes to different servers → session may be lost.

With sticky sessions: User always hits the same server, so their cart stays intact.

🛠️ How to Enable:
In Target Group settings, enable:

“Stickiness: Enabled”

Choose “Duration” (like 5 minutes)

⚠️ Interview Tip:
Q: What’s the downside of sticky sessions?

A: If the target fails, the session may break. Also, it can cause uneven traffic if some users stay connected longer.

3️⃣ SNI (Server Name Indication) — 🔒 Multiple SSLs on one ALB
✅ What is SNI?
SNI stands for Server Name Indication. It’s a TLS/SSL extension that allows one Load Balancer (or server) to use multiple SSL certificates for different domain names on the same listener.

📦 Real Example:
You host:

app.example.com

admin.example.com

Both use HTTPS and are served by the same ALB.

Using SNI, ALB can serve different certificates:

app.example.com → Cert A

admin.example.com → Cert B

All on port 443, same listener!

🔐 Without SNI?
Without SNI, you’re limited to one SSL certificate per listener → hard to host multiple HTTPS sites.

💬 Interview Tip:
Q: Why is SNI important in ALB?

A: Because ALB supports hosting multiple domains with HTTPS on a single listener, saving cost and complexity.

🔁 Summary Table
Term	Simple Meaning	Real Use
Target Group	Group of backend services	Tells ELB where to send traffic
Sticky Session	Keep user with same server	Needed for stateful apps (e.g., shopping cart)
SNI	Multiple SSL certs on same listener	Host multiple HTTPS domains on one ALB
