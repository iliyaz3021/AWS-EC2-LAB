# AWS Services: EC2,SECURITY GROUPS,LOAD BALANCING,AUTO SCALLING

---

##  Phase 1: Initial EC2 Setup & Configuration

**AWS EC2:** `EC2 Dashboard` ➔ `Launch Instance`

* **Instance Details:** * Name: `MyWebServer`
    * AMI: `Amazon Linux 2023` (Architecture: x86_64)
    * Type: `t2.micro`
* **Security Rules:**
    *  `SSH` (Port 22) ➔ Restricted to `My IP`
    *  `HTTP` (Port 80) ➔ Open to `0.0.0.0/0` (Anywhere)

---

##  Phase 2: Manual Web Server Bootstrapping
After SSH-ing into the instance, I ran the following commands to transform the Linux instance into a functional web server:

1. `sudo yum update -y` ➔ **Refreshed system repositories**
2. `sudo yum install httpd -y` ➔ **Installed Apache Web Server**
3. `sudo systemctl start httpd` ➔ **Initiated the service**
4. `sudo systemctl enable httpd` ➔ **Ensured persistence on reboot**
5. `echo "Hello from $(hostname)" | sudo tee /var/www/html/index.html` ➔ **Created a simple page to show the hostname**

---

##  Phase 3: Creating the image (AMI)
To ensure every new instance in the scaling group is identical, I created a custom AMI.

* **Step:** `Instances` ➔ `Actions` ➔ `Image and templates` ➔ `Create Image`
* **Next:** I created a **Launch Template** named `MyTemplate`, linking it to `MyWebAMI` so the Auto Scaling Group knows what to launch.

---

## Phase 4: Load Balancing & Auto Scaling Implementation
This is where the high-availability logic was applied.

### 1. Target Group
* **Path:** `Target Groups` ➔ `Create` ➔ `Instances`
* **Config:** Protocol: `HTTP` | Port: `80` | Name: `MyTargetGroup`

### 2. Application Load Balancer (ALB)
* **Path:** `Load Balancers` ➔ `Create ALB` ➔ `Internet-facing`
* **Network:** Selected **2 Subnets** across different Availability Zones (AZs) for redundancy.
* **Listener:** Forwarding all Port 80 traffic ➔ `MyTargetGroup`.

### 3. Auto Scaling Group (ASG)
* **Path:** `Auto Scaling Groups` ➔ `Create ASG` ➔ linked to `MyTemplate`.
* **Capacity Settings:** * Desired: `2` (Always keep 2 running)
    * Min: `2` | Max: `4`
* **Dynamic Scaling:** Enabled `Target Tracking Policy` ➔ **Scale if Avg CPU > 50%**.

---

##  Phase 5: Validation & Stress Testing
I performed two tests to ensure the architecture works as intended:

* **LB Check:** Opened the `ALB DNS Name` in a browser. Every time I refreshed, the **hostname changed**. This proves the Load Balancer is successfully alternating traffic between instances in different AZs.
  
* **Scaling Check:** I installed a stress tool on one instance to simulate high traffic:
    * `sudo yum install stress -y`
    * `stress --cpu 2 --timeout 300`
