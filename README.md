# Lab M2.01 - Launch Your First EC2 Instance

**Repository:** [https://github.com/cloud-engineering-bootcamp/ce-lab-launch-first-ec2](https://github.com/cloud-engineering-bootcamp/ce-lab-launch-first-ec2)

**Activity Type:** Individual  
**Estimated Time:** 45-60 minutes

## Learning Objectives

- [ ] Launch an EC2 instance using the AWS Console
- [ ] Configure a security group with appropriate rules
- [ ] Create and secure an SSH key pair
- [ ] Connect to your EC2 instance via SSH
- [ ] Verify the instance is running and accessible

## Prerequisites

- [ ] AWS account with appropriate IAM permissions (EC2, VPC)
- [ ] Terminal application (built-in on Mac/Linux, or Git Bash on Windows)
- [ ] Basic understanding of EC2 concepts from lessons

---

## Introduction

Your first hands-on experience with cloud compute! You'll launch a virtual server in AWS, configure its security, and connect to it remotely. This is the foundation of cloud engineering - everything else builds on these skills.

## Scenario

You're a cloud engineer at a startup, and your team needs a Linux server for:
- Hosting development tools
- Running automated scripts
- Testing deployment configurations

Your task is to launch an EC2 instance, secure it properly, and verify you can connect and manage it.

---

## Your Task

**What you'll create:**
- 1 EC2 instance (t3.micro - free tier eligible)
- 1 security group with proper SSH access
- 1 SSH key pair for secure access
- Successful SSH connection to your instance

**Success criteria:**
- [ ] EC2 instance in "running" state
- [ ] Security group allows SSH only from your IP
- [ ] SSH key pair created and downloaded
- [ ] Successfully connect via SSH
- [ ] Run basic commands on the instance
- [ ] Take screenshots for submission

**Time limit:** 45-60 minutes

---

## Step-by-Step Instructions

### Step 1: Find Your IP Address

Before creating security rules, you need your public IP.

**Option 1: Web browser**
- Visit: https://whatismyipaddress.com/

**Option 2: Terminal**
```bash
curl ifconfig.me
# or
curl icanhazip.com
```

**Write down your IP: 143.179.136.22 (you'll need this!)

---

### Step 2: Create an SSH Key Pair

**Via AWS Console:**

1. Navigate to **EC2 Dashboard**
   - AWS Console → Services → EC2

2. In left sidebar: **Network & Security** → **Key Pairs**

3. Click **"Create key pair"**

4. **Configure key pair:**
   - **Name:** `bootcamp-week2-key`
   - **Key pair type:** RSA
   - **Private key file format:** 
     - `.pem` (for Mac/Linux)
     - `.ppk` (for Windows with PuTTY)

5. Click **"Create key pair"**

6. **⚠️ IMPORTANT:** The private key file will download automatically
   - **This is your ONLY chance to download it!**
   - Save it to a secure location (e.g., `~/.ssh/`)

7. **Secure the key file (Mac/Linux):**
   ```bash
   # Move to ssh directory
   mv ~/Downloads/bootcamp-week2-key.pem ~/.ssh/
   
   # Set restrictive permissions
   chmod 400 ~/.ssh/bootcamp-week2-key.pem
   ```

**Expected outcome:** You have a private key file saved securely on your computer.

---

### Step 3: Create a Security Group

1. In EC2 Dashboard left sidebar: **Network & Security** → **Security Groups**

2. Click **"Create security group"**

3. **Basic details:**
   - **Security group name:** `week2-web-server-sg`
   - **Description:** `Security group for Week 2 lab - allows SSH and HTTP`
   - **VPC:** Select default VPC

4. **Inbound rules:**
   
   Click **"Add rule"** for each:
   
   **Rule 1 - SSH:**
   - **Type:** SSH
   - **Protocol:** TCP (auto-filled)
   - **Port range:** 22 (auto-filled)
   - **Source:** My IP (will auto-detect) OR Custom → `YOUR_IP/32`
   - **Description:** `SSH from my IP`
   
   **Rule 2 - HTTP:**
   - **Type:** HTTP
   - **Protocol:** TCP
   - **Port range:** 80
   - **Source:** Anywhere-IPv4 (0.0.0.0/0)
   - **Description:** `Public HTTP access`

5. **Outbound rules:**
   - Leave default (All traffic to 0.0.0.0/0)

6. **Tags:**
   - Key: `Name`, Value: `week2-web-server-sg`
   - Key: `Environment`, Value: `Development`

7. Click **"Create security group"**

**Expected outcome:** A new security group that allows SSH from your IP and HTTP from anywhere.

---

### Step 4: Launch EC2 Instance

1. **EC2 Dashboard** → Click **"Launch Instance"**

2. **Name and tags:**
   - **Name:** `week2-web-server`
   - **Additional tags:**
     - `Environment`: `Development`
     - `Project`: `CloudBootcamp`

3. **Application and OS Images (AMI):**
   - **Quick Start** → **Amazon Linux**
   - Select: **Amazon Linux 2023 AMI** (Free tier eligible)
   - Architecture: **64-bit (x86)**

4. **Instance type:**
   - Select: **t3.micro** (Free tier eligible)
   - 1 vCPU, 1 GB Memory

5. **Key pair:**
   - Select: **bootcamp-week2-key** (created in Step 2)

6. **Network settings:**
   - Click **"Edit"**
   - **VPC:** (keep default)
   - **Subnet:** (keep default - will auto-assign public subnet)
   - **Auto-assign public IP:** **Enable**
   - **Firewall (security groups):**
     - Select **"Select existing security group"**
     - Choose: **week2-web-server-sg**

7. **Configure storage:**
   - **Size:** 8 GB (default)
   - **Volume type:** gp3 (General Purpose SSD)
   - **Delete on termination:** Yes (checked)
   - **Encrypted:** No (for this lab)

8. **Advanced details** → **User data:**
   
   Paste this script (installs Apache web server):
   ```bash
   #!/bin/bash
   # Update system
   yum update -y
   
   # Install Apache web server
   yum install -y httpd
   
   # Start Apache
   systemctl start httpd
   systemctl enable httpd
   
   # Create a simple webpage
   echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
   echo "<p>Instance ID: $(ec2-metadata --instance-id | cut -d ' ' -f 2)</p>" >> /var/www/html/index.html
   echo "<p>Availability Zone: $(ec2-metadata --availability-zone | cut -d ' ' -f 2)</p>" >> /var/www/html/index.html
   ```

9. **Summary:**
   - Review your configuration
   - **Number of instances:** 1

10. Click **"Launch instance"**

11. **Wait for launch:**
    - Status will show "Launching"
    - Wait ~1-2 minutes
    - Refresh until status shows "Running"

**Expected outcome:** Your EC2 instance is running with a public IP address.

---

### Step 5: Connect via SSH

1. **Find your instance's public IP:**
   - EC2 Dashboard → Instances
   - Select your instance
   - Copy the **"Public IPv4 address"** (e.g., 54.123.45.67)

2. **Open your terminal**

3. **Connect via SSH:**
   ```bash
   ssh -i ~/.ssh/bootcamp-week2-key.pem ec2-user@YOUR_PUBLIC_IP
   ```
   
   Replace `YOUR_PUBLIC_IP` with actual IP address.

4. **First connection:**
   - You'll see a message: `The authenticity of host '54.123.45.67' can't be established.`
   - Type `yes` and press Enter
   - This adds the host to your known_hosts file

5. **You're in!** You should see a prompt like:
   ```
   [ec2-user@ip-10-0-1-50 ~]$
   ```

**Expected outcome:** You're connected to your EC2 instance via SSH.

---

### Step 6: Explore Your Instance

Run these commands to learn about your instance:

```bash
# 1. Check OS information
cat /etc/os-release

# 2. Check system resources
free -h        # Memory
df -h          # Disk space
nproc          # Number of CPUs

# 3. Check running processes
top            # Press 'q' to quit

# 4. Check if Apache is running
sudo systemctl status httpd

# 5. View the instance metadata
ec2-metadata --all

# 6. Check network configuration
ip addr show
```

**Take notes on what you find!**

---

### Step 7: Test the Web Server

1. **In your web browser**, visit:
   ```
   http://54.163.11.40/
   ```
   
   (Use the same IP you used for SSH)

2. **You should see:**
   - A webpage saying "Hello from ip-172-31-18-190.ec2.internal"
   - Instance ID: i-0f1913ce72c45fb6b
   - Availability Zone: us-east-1c

3. **Take a screenshot** of the webpage

**Expected outcome:** Your web server is publicly accessible.

---

### Step 8: Verify Security

Test that your security group is working correctly:

1. **SSH should work from your IP:**
   ```bash
   ssh -i ~/.ssh/bootcamp-week2-key.pem ec2-user@YOUR_PUBLIC_IP
   # Should connect successfully ✅
   ```

2. **Try ping (should timeout):**
   ```bash
   ping YOUR_PUBLIC_IP
   # Should timeout - ICMP not allowed ❌
   ```

3. **HTTP should work:**
   ```bash
   curl /http://54.163.11.40/
   # Should return HTML ✅
   ```

---

### Step 9: Check Instance Details via AWS CLI (Bonus)

If you have AWS CLI installed:

```bash
# Get instance information
aws ec2 describe-instances \
    --instance-ids YOUR_INSTANCE_ID \
    --query 'Reservations[0].Instances[0].[InstanceId,InstanceType,State.Name,PublicIpAddress]' \
    --output table

# Get security group rules
aws ec2 describe-security-groups \
    --group-ids YOUR_SECURITY_GROUP_ID \
    --output table
```

---

## 📤 What to Submit

**Submission Type:** GitHub Repository

Create a **public** GitHub repository called `ce-lab-launch-ec2-instance` with:

### Required Files

**1. README.md** - Include:
- Your name and date
- Lab description
- Instance details (ID, type, IP)
- Security group configuration
- Screenshots with captions
- Challenges you faced and how you solved them

**2. Screenshots folder** - Include:
- `01-instance-running.png` - EC2 console showing running instance
- `02-security-group-rules.png` - Security group inbound rules
- `03-ssh-connection.png` - Terminal showing successful SSH connection
- `04-web-server-browser.png` - Web browser showing your webpage
- `05-instance-details.png` - Instance details page

**3. security-group-rules.txt** - Copy/paste:
```
Inbound Rules:
Type: SSH  | Protocol: TCP | Port: 22 | Source: 143.179.136.22/32 | Description: SSH from my IP
Type: HTTP | Protocol: TCP | Port: 80 | Source: 0.0.0.0/0        | Description: Public HTTP access

Outbound Rules:
Type: All traffic | Protocol: All | Port: All | Destination: 0.0.0.0/0 | Description: (default)
```

**4. instance-info.txt** - Run on instance and save output:
```bash
echo "=== System Information ===" > instance-info.txt
cat /etc/os-release >> instance-info.txt
echo -e "\n=== Resources ===" >> instance-info.txt
free -h >> instance-info.txt
df -h >> instance-info.txt
echo -e "\n=== Network ===" >> instance-info.txt
ip addr show >> instance-info.txt
```

### Repository Structure
```
ce-lab-launch-ec2-instance/
├── README.md
├── screenshots/
│   ├── 01-instance-running.png
│   ├── 02-security-group-rules.png
│   ├── 03-ssh-connection.png
│   ├── 04-web-server-browser.png
│   └── 05-instance-details.png
├── security-group-rules.txt
└── instance-info.txt
```

---

## Cleanup (Important!)

To avoid charges, stop or terminate your instance:

### Option 1: Stop (Pause)
```
EC2 Console → Instances → Select instance → 
Instance state → Stop instance
```
- Stops compute charges
- Preserves data
- Can restart later
- Still pay for EBS storage (~$0.80/month for 8GB)

### Option 2: Terminate (Delete)
```
EC2 Console → Instances → Select instance → 
Instance state → Terminate instance
```
- Stops all charges
- **Permanently deletes** instance and data
- Cannot be recovered

**⚠️ For this lab:** You can stop the instance after submission. We'll use it in later labs.

---

## Bonus Challenges

### Challenge 1: Multiple Key Pairs

- Create a second key pair
- Try to connect with wrong key
- Document what error you see

### Challenge 2: Change Instance Type

- Stop your instance
- Change instance type to `t3.small`
- Start and verify it works
- Document CPU/memory differences

### Challenge 3: Add HTTPS Support

- Add HTTPS (port 443) to security group
- Install certbot and create self-signed certificate
- Test HTTPS access

### Challenge 4: Instance Metadata Service

Query instance metadata from inside the instance:

```bash
# Get instance ID
curl http://169.254.169.254/latest/meta-data/instance-id

# Get availability zone
curl http://169.254.169.254/latest/meta-data/placement/availability-zone

# Get security groups
curl http://169.254.169.254/latest/meta-data/security-groups
```

Document what you discover!

---

## Common Issues & Solutions

### Issue 1: "Permission denied (publickey)"

**Cause:** Wrong key file or wrong user name

**Solution:**
```bash
# Check username - Amazon Linux uses ec2-user
ssh -i ~/.ssh/bootcamp-week2-key.pem ec2-user@YOUR_IP

# Verify key permissions
ls -l ~/.ssh/bootcamp-week2-key.pem
# Should show: -r--------  (400 permissions)

# Fix if needed
chmod 400 ~/.ssh/bootcamp-week2-key.pem
```

### Issue 2: "Connection timed out"

**Causes:**
1. Security group doesn't allow SSH from your IP
2. Instance not in running state
3. Wrong IP address

**Solution:**
1. Check security group has SSH rule for your IP
2. Verify instance state is "running"
3. Use Public IPv4 address (not private)

### Issue 3: "Web page not loading"

**Causes:**
1. Apache not installed/running
2. Security group missing HTTP rule
3. Using HTTPS instead of HTTP

**Solution:**
```bash
# SSH into instance and check Apache
sudo systemctl status httpd

# Restart if needed
sudo systemctl restart httpd

# Verify security group allows port 80 from 0.0.0.0/0
```

---

## Learning Reflections

After completing this lab, reflect on:

1. **What surprised you most** about launching a cloud server?
2. **How is this different** from setting up a physical server or local VM?
3. **What security concerns** did you need to consider?
4. **How long** would this process take with traditional infrastructure?

---

## Additional Exploration

Want to learn more? Try these:

1. **Install additional software:**
   ```bash
   # Install Git
   sudo yum install -y git
   
   # Install Python
   sudo yum install -y python3
   
   # Install Node.js
   curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
   sudo yum install -y nodejs
   ```

2. **Monitor your instance:**
   - EC2 Console → Select instance → Monitoring tab
   - Observe CPU, network, disk metrics

3. **Create an AMI:**
   - Select instance → Actions → Image and templates → Create image
   - Use this AMI to launch identical instances

4. **Explore instance user data:**
   - Select instance → Actions → Instance settings → Edit user data
   - See the script that ran at launch

---

## Reference Commands

### Essential SSH Commands
```bash
# Connect to instance
ssh -i ~/.ssh/bootcamp-week2-key.pem ec2-user@PUBLIC_IP

# Copy file to instance
scp -i ~/.ssh/bootcamp-week2-key.pem file.txt ec2-user@PUBLIC_IP:~/

# Copy file from instance
scp -i ~/.ssh/bootcamp-week2-key.pem ec2-user@PUBLIC_IP:~/file.txt ./

# SSH with verbose output (for debugging)
ssh -v -i ~/.ssh/bootcamp-week2-key.pem ec2-user@PUBLIC_IP
```

### System Commands on Instance
```bash
# Check system info
uname -a                 # Kernel version
cat /etc/os-release      # OS details
uptime                   # System uptime

# Check resources
free -h                  # Memory usage
df -h                    # Disk usage
top                      # Process monitoring
htop                     # Better process monitoring (if installed)

# Check network
ip addr show             # IP addresses
ip route show            # Routing table
ss -tuln                 # Listening ports

# Service management
sudo systemctl status httpd    # Check Apache status
sudo systemctl start httpd     # Start Apache
sudo systemctl restart httpd   # Restart Apache
sudo systemctl enable httpd    # Enable on boot
```

---

## Submission Checklist

Before submitting, verify:

- [ ] GitHub repository created and public
- [ ] README.md is complete and well-formatted
- [ ] All 5 required screenshots included
- [ ] Screenshots are clear and properly named
- [ ] security-group-rules.txt contains your actual rules
- [ ] instance-info.txt contains output from your instance
- [ ] Repository URL submitted to platform
- [ ] Instance stopped or terminated to avoid charges

---

## Grading Rubric

| Criteria | Points | Requirements |
|----------|--------|--------------|
| **Instance Launch** | 25 | Instance running, correct type (t3.micro), proper tags |
| **Security Group** | 25 | SSH restricted to your IP, HTTP from anywhere, proper descriptions |
| **SSH Connection** | 20 | Successfully connected, proof via screenshot |
| **Web Server** | 15 | Apache running, accessible via browser |
| **Documentation** | 15 | Complete README, all screenshots, clear explanations |
| **Total** | **100** | |

---

## Time Tracking

Estimate how long each section took:

- [ ] Creating key pair: _____ minutes
- [ ] Creating security group: _____ minutes
- [ ] Launching instance: _____ minutes
- [ ] Connecting via SSH: _____ minutes
- [ ] Testing and screenshots: _____ minutes
- [ ] Documentation: _____ minutes

**Total time:** _____ minutes

---

## Next Steps

After completing this lab:
- Keep your instance running (or stopped) for tomorrow's labs
- You'll build on this foundation by installing more software
- We'll explore advanced networking and security

**Congratulations on launching your first EC2 instance!** 🎉
