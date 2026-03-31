# AWS Services: EC2,SECURITY GROUPS,LOAD BALANCING,AUTO SCALLING,IAM,S3, full stack app with jenkins practice, Practiced redis and understanding between services

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
