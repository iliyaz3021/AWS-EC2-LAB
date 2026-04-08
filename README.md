# AWS Services: EC2,SECURITY GROUPS,LOAD BALANCING,AUTO SCALLING,IAM,S3, full stack app with jenkins practice, Practiced redis and understanding between services, VPC

---

## 1: Laucned EC2 Setup & Configuration

**AWS EC2:** `EC2 Dashboard` ➔ `Launch Instance`

* **Instance Details:** * Name: `MyWebServer`
    * AMI: `Amazon Linux 2023` (Architecture: x86_64)
    * Type: `t2.micro`
* **Security Rules:**
    *  `SSH` (Port 22) ➔ Restricted to `My IP`
    *  `HTTP` (Port 80) ➔ Open to `0.0.0.0/0` (Anywhere)

---

##  2: Defined Web Server On EC2 Server
After SSH-ing into the instance, I ran the following commands to transform the Linux instance into a functional web server:

1. `sudo yum update -y` ➔ **Refreshed system repositories**
2. `sudo yum install httpd -y` ➔ **Installed Apache Web Server**
3. `sudo systemctl start httpd` ➔ **Initiated the service**
4. `sudo systemctl enable httpd` ➔ **Ensured persistence on reboot**
5. `echo "Hello from $(hostname)" | sudo tee /var/www/html/index.html` ➔ **Created a simple page to show the hostname**

---

## 3: Created the image (AMI) for copy of server
To ensure every new instance in the scaling group is identical, I created a custom AMI.

* **Step:** `Instances` ➔ `Actions` ➔ `Image and templates` ➔ `Create Image`
* **Next:** I created a **Launch Template** named `MyTemplate`, linking it to `MyWebAMI` so the Auto Scaling Group knows what to launch.

---

##  4: Launched Load Balancing & Auto Scaling to manage traffic load
This is where the high-availability logic was applied.

### 5. Created Target Group To SetUp Load Balancer
* **Path:** `Target Groups` ➔ `Create` ➔ `Instances`
* **Config:** Protocol: `HTTP` | Port: `80` | Name: `MyTargetGroup`

### 6. Created Application Load Balancer (ALB) for Distributing the Traffic Load To Servers
* **Path:** `Load Balancers` ➔ `Create ALB` ➔ `Internet-facing`
* **Network:** Selected **2 Subnets** across different Availability Zones (AZs) for redundancy.
* **Listener:** Forwarding all Port 80 traffic ➔ `MyTargetGroup`.

### 7. Auto Scaling Group (ASG) For Increasing and Decreasing the Servers Based On CPU Utilization
* **Path:** `Auto Scaling Groups` ➔ `Create ASG` ➔ linked to `MyTemplate`.
* **Capacity Settings:** * Desired: `2` (Always keep 2 running)
    * Min: `2` | Max: `4`
* **Dynamic Scaling:** Enabled `Target Tracking Policy` ➔ **Scale if Avg CPU > 50%**.

---

##   8: Validation & Stress Testing for checking load Balancing and Auto Scalling
I performed two tests to ensure the architecture works as intended:

* **LB Check:** Opened the `ALB DNS Name` in a browser. Every time I refreshed, the **hostname changed**. This proves the Load Balancer is successfully alternating traffic between instances in different AZs.
  
* **Scaling Check:** I installed a stress tool on one instance to simulate high traffic:
    * `sudo yum install stress -y`
    * `stress --cpu 2 --timeout 300`
 

---

# 9: IAM Setup (Users, Roles, Permissions)

**AWS IAM:** `IAM Dashboard` ➔ `Users / Roles`

---

## Created IAM User

* Path: `IAM` ➔ `Users` ➔ `Create User`
* Username: `DevUser`
* Access:

  * Console Access
  * Programmatic Access

---

## Assigned Permissions

* Attached policy: `AmazonS3FullAccess`
* Used only required permissions (basic security practice)

---

## Created IAM Role for EC2

* Path: `IAM` ➔ `Roles` ➔ `Create Role`
* Use case: `EC2`
* Policy: `AmazonS3ReadOnlyAccess`

👉 This helps EC2 access S3 without using access keys

---

## Attached Role to EC2

* Path: `EC2` ➔ `Instances` ➔ `Actions` ➔ `Security` ➔ `Modify IAM Role`
* Selected role: `EC2-S3-Access-Role`

---

# 10: S3 Setup (Storage & Static Website)

**AWS S3:** `S3 Dashboard` ➔ `Create Bucket`

---

## Created S3 Bucket

* Bucket Name: `my-web-assets-bucket`
* Region: Same as EC2
* Public Access: Disabled block (for testing)

---

## Uploaded Files

* Uploaded HTML file and sample assets
* Checked object URL in browser

---

## Enabled Static Website Hosting

* Path: `Bucket` ➔ `Properties` ➔ `Static Website Hosting`
* Index file: `index.html`

👉 Website was accessible using S3 endpoint

---

## Added Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-web-assets-bucket/*"
    }
  ]
}
```

---

## Accessed S3 from EC2

* Verified using IAM role (no access keys used)

```bash
aws s3 ls
aws s3 cp s3://my-web-assets-bucket/index.html .
```
-----------------------------------------------------------------------------------------------------------------------------------
# Hands On Practice


---

# 🔐 6: IAM Setup (User & Role Configuration)

**AWS IAM:** `IAM Dashboard`

---

## Created IAM User

* Path: `IAM` ➔ `Users` ➔ `Create User`
* Username: `DevUser`
* Access:

  * Console access
  * Programmatic access

---

## Assigned Permissions

* Attached policy: `AmazonS3FullAccess`

---

## Created IAM Role for EC2

* Path: `IAM` ➔ `Roles` ➔ `Create Role`

* Use case: `EC2`

* Initially attached:

  * `AmazonS3ReadOnlyAccess`

---

## Attached Role to EC2

* Path: `EC2` ➔ `Instances` ➔ `Modify IAM Role`
* Role: `EC2-S3-Access-Role`

👉 EC2 can now access AWS services securely

---

## Updated IAM Role (During Testing)

While uploading file from EC2 to S3, I got **AccessDenied error**

**Reason:**

* Role had only ReadOnly access

**Fix:**

* Added permission:

  * `AmazonS3FullAccess`

👉 This allowed uploading files to S3

---

# 🪣 7: S3 Setup (Storage & Static Website)

**AWS S3:** `S3 Dashboard`

---

## Created S3 Bucket

* Bucket Name: `my-web-assets-bucket012345`
* Region: Same as EC2
* Disabled block public access (for testing)

---

## Uploaded Files

* Uploaded `index.html`

```html
<h1>Hello from S3</h1>
```

---

## Enabled Static Website Hosting

* Path: `Bucket` ➔ `Properties` ➔ `Static Website Hosting`
* Index document: `index.html`

👉 Accessed website using S3 endpoint URL

---

## Added Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-web-assets-bucket012345/*"
    }
  ]
}
```

---

## Accessed S3 from EC2

Verified connection using IAM role:

```bash
aws s3 ls
aws s3 cp s3://my-web-assets-bucket012345/index.html .
```

👉 Successfully downloaded file

---

## Uploaded File from EC2 to S3

```bash
echo "Hello from EC2" > test.txt
aws s3 cp test.txt s3://my-web-assets-bucket012345/
```

👉 File uploaded successfully after updating IAM permissions

---


---

## What I Did

I set up an automated deployment pipeline for our fullstack app using Jenkins and Docker.

**Here's how it works:**

I push code to GitHub → Jenkins automatically detects the push → Jenkins builds a Docker image of the app → Jenkins runs the container using docker-compose → App is up and running.

**What this means:**

Before: Every time I changed code, I had to manually build and run the app again.

After: Now I just push to GitHub, and Jenkins handles everything automatically. No manual work needed.

**Tools I used:**
- GitHub → to store our code
- Jenkins → to automate the build and deploy process
- Docker → to package and run the app consistently

**The flow in one line:**

GitHub Push → Jenkins Triggers → Docker Build → Container Runs → App Deployed

---


# System Flow (Frontend → Backend → Redis)
 ## Frontend to Backend

User Action → React (Frontend) → API Call (Axios) → Flask Backend

## First Login 

Frontend → /login → Backend → Check Redis ❌ (MISS)
→ Query Database → Verify Password (bcrypt)
→ Store User in Redis (SETEX)
→ Response to Frontend
→ ⏱️ Latency: High 

## Second Login (With Cache)

Frontend → /login → Backend → Check Redis ✅ (HIT)
→ Get User from Redis → Verify Password 
→ Response to Frontend
→ ⏱️ Latency: Medium 

---



# vpc
## public subnet ->web server( Accessible from internet)
## Private Subnet -> Data Base server(No internet Access)
## NAT Gateway   -> Allows orivate instance to download updates securly


## Step 1: Created VPC with advanced settings
. VPC name → "my-app-vpc"
. CIDR → 10.0.0.0/16
. Availability Zones → 2 (for high availability)
. Public subnets → 2 (10.0.1.0/24, 10.0.2.0/24)
.  Private subnets → 2 (10.0.3.0/24, 10.0.4.0/24)
. NAT Gateway → 1 per AZ 
. IGW → Auto-created 

## Step 2: Launch Public web server
. Name → "Public-Web-Server"
, Subnet → public-subnet-az1
. Auto-assign public IP → ENABLE
. Security group → Allow HTTP(80) from 0.0.0.0/0
. Security group → Allow SSH(22) from My IP
. User data → httpd installation script


## Step 3: Launched Private Database Server
. Name → "Private-DB-Server"
. Subnet → private-subnet-az1
. Auto-assign public IP → DISABLE (CRITICAL)
. Security group → No direct SSH from internet


## Step 4: Test public sever
. copy public ip of public webserver
. Open browser → http://<public-ip>
. Web page loads

## Step 5: Test Private Server's Internet Access
. Go to EC2 Console → Select Private-DB-Server
. Click "Connect" → Session Manager tab
. Run: curl -s http://icanhazip.com
. Output shows NAT Gateway's IP (not instance's IP)
        (Proves private instance uses NAT to reach internet)

---
# AWS Practice Summary
### IAM (Identity and Access Management)
### Users & Groups

Create User → Create Group → Assign Permissions to Group → Add User to Group → User inherits permissions

### Policies

Write JSON Policy → Attach to Group → Control access (S3 ReadOnly, etc.)

### Key Concepts

Least Privilege → Give only required access
Group-based Access → Easy to manage multiple users
Permissions = Combined if user in multiple groups

## S3 (Simple Storage Service)

Create Bucket → Upload Files → Enable Static Hosting → Access via URL

### Security

S3 Private by default → Add Bucket Policy for access → Control public/private access

### IAM + S3 Integration

Create Policy → Attach to Group → Add User → Access S3 based on permissions

## AWS CloudFormation (Infrastructure as Code)

Write YAML Template → Create Stack → AWS creates resources automatically

Example Flow

Write Template → Upload → Create Stack → Resources Created → Update/Delete Stack

### Key Concepts

Stack = Collection of resources
Template = Code (YAML/JSON)
Automation → No manual setup

### EC2 + AMI

Find AMI ID → Launch EC2 Instance → Use in CloudFormation

### AMI

AMI = Pre-configured OS template
Region-specific → Different AMI per region
Use SSM → Get latest AMI automatically


---

## CloudFront

Amazon CloudFront is a Content Delivery Network (CDN) that distributes content through global edge locations to reduce latency and improve performance. It caches content closer to users, ensuring faster delivery and better user experience.

 ## Route 53

Amazon Route 53 is a scalable Domain Name System (DNS) service that translates domain names into IP addresses. It also provides routing policies and health checks to ensure high availability and reliable traffic management.

## CI/CD Basics

CI/CD (Continuous Integration and Continuous Delivery/Deployment) is a DevOps practice that automates the process of building, testing, and deploying applications. CI focuses on integrating code changes and running tests, while CD ensures that the application is delivered or deployed to production efficiently and reliably.
