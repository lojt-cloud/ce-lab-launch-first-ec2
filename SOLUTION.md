# Lab Solution
# Lab M2.01 - Launch Your First EC2 Instance

**Student Name:** Balint Lojt
**Date:** [today's date]

---

## Instance Details

**Instance ID: i-0f1913ce72c45fb6b
**Instance Type: t3.micro
**Public IP: 54.163.11.40
**Region/Availability Zone:**
us-east-1 / us-east-1c
---

## Security Group Configuration

**Security Group Name:** week2-web-server-sg

**Inbound Rules:**

Type: SSH  | Protocol: TCP | Port: 22 | Source: 143.179.136.22/32 | Description: SSH from my IP
Type: HTTP | Protocol: TCP | Port: 80 | Source: 0.0.0.0/0        | Description: Public HTTP access

**Outbound Rules:**

Type: All traffic | Protocol: All | Port: All | Destination: 0.0.0.0/0 | Description: (default)


## Security Verification

- SSH:  Successfully connected from my IP
- HTTP:  Web page loads correctly via browser and curl
- Ping (ICMP):  No response (times out) - confirms security group only
  allows the explicitly configured SSH and HTTP traffic, nothing else

## Screenshots

**1. Instance Running:**
![Instance Running](screenshots/01-instance-running.png)

**2. Security Group Rules:**
![Security Group](screenshots/02-security-group-rules.png)

**3. SSH Connection:**
![SSH Connection](screenshots/03-ssh-connection.png)

**4. Web Server (Browser):**
![Web Server](screenshots/04-web-server-browser.png)

**5. Instance Details:**
![Instance Details](screenshots/05-instance-details.png)

---

## Challenges Faced and Solutions
1.
When creating the security group, the "Create security group" button appeared
unresponsive - no error shown. After checking the VPC dropdown specifically,
it showed "No VPC available." Switching regions didn't resolve it. Checking
the VPC Console directly confirmed there were genuinely no VPCs in that
region (not a UI bug, and no account limit reached). Fixed by going to
VPC Console → Your VPCs → Actions → Create default VPC, which automatically
provisioned a complete default VPC setup. After that, the security group
creation worked immediately.

2.
AWS CLI commands (describe-security-groups) initially failed with credential
and "security group not found" errors. First issue was expired/missing local
CLI credentials - fixed with `aws configure`. Second issue was a region
mismatch: my CLI's default region is eu-north-1, but this lab's resources
were created in us-east-1 (from the earlier VPC fix). Resolved by explicitly
specifying `--region us-east-1` on the command. This is the same category of
issue I hit earlier this week with EC2 tagging - for future me: always
check `aws configure get region` first whenever a resource "isn't found"
unexpectedly.
---

## Learning Reflections

### What surprised you most about launching a cloud server?
It took much less time than I expected once the correct steps were clear.
from nothing to a running, publicly accessible web server in under an hour,
including troubleshooting a VPC issue along the way. I was also surprised
by how much attention security requires even for a simple lab like restricting
SSH to only my IP (rather than 0.0.0.0/0) matters because leaving port 22
open to the entire internet would let anyone attempt to brute-force or
exploit the server, not just me.
### How is this different from setting up a physical server or local VM?
Configuration is much faster. I just select the specs I want (instance
type, OS) rather than physically installing hardware or manually setting
up a local VM's virtual hardware. The user data script also let me pre-load
Apache so the server was fully configured and serving content the moment it
booted, no manual install needed afterward.

Beyond speed, there's no physical hardware to own or maintain at all.
With a physical server I'd need to buy, rack, and power actual equipment, and
even a local VM still depends on hardware I own and maintain. With EC2, if
something's wrong I can just terminate the instance and launch a fresh one
in minutes, and I only pay for the hours it's actually running, rather than
a large upfront cost or a machine sitting idle regardless of use.

### What security concerns did you need to consider?

The main concern was correctly scoping each port to who should actually
have access. SSH (port 22) needed to be restricted to only my own IP
address (using /32), since leaving it open to 0.0.0.0/0 would expose
administrative access to anyone on the internet. HTTP (port 80) was
intentionally left open to everyone, since a web server is meant to be
publicly accessible - different purposes need different exposure levels.

I also had to handle the SSH private key file securely - setting it to
read-only permissions (chmod 400) and making sure it never ended up
committed to the git repository, since it's a real credential that would
let anyone with it access my instance directly.

Beyond just configuring the rules, I also verified them - confirming SSH
and HTTP worked as expected, and that ping (ICMP) was correctly blocked
since it wasn't explicitly allowed, proving the security group wasn't
accidentally more permissive than intended.

### How long would this process take with traditional infrastructure?
Hours to days, not minutes. Physical hardware would need to be purchased,
racked, and cabled, or even a local VM's host machine still needs OS
installation and setup. Then manually configuring all the network ports
and firewall rules, testing that connectivity works, installing the actual
software (Apache in this case), testing again, and checking everything is
updated to the latest version. Each of those steps that took seconds in the
AWS Console (selecting an instance type, running a startup script) would be
a manual, time-consuming task with traditional infrastructure.

---

## Time Tracking

- Creating key pair: 5 minutes
- Creating security group: 15 minutes (had issue that I explained in the challanges part)
- Launching instance: 1 minutes
- Connecting via SSH: 2 minutes
- Testing and screenshots: 20 minutes
- Documentation: 1 hour 10  minutes

**Total time:** 1 hour 53 minutes