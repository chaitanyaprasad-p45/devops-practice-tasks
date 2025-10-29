# AWS Tasks Solutions

This document contains detailed console-based solutions for all 50 AWS practice tasks.

## Basic Level Solutions (Tasks 1-15)

### Task 1: Account Setup
**Create a new AWS account and configure billing alerts for $10, $50, and $100.**

**Console-Based Solution:**

**Step 1: Create AWS Account**
1. Navigate to https://aws.amazon.com
2. Click "Create an AWS Account" button
3. Enter email address and account name
4. Choose "Personal" account type
5. Fill in contact information
6. Enter payment method (credit card)
7. Verify phone number with SMS/call
8. Select Basic Support plan (free)
9. Complete account creation

**Step 2: Enable Billing Alerts**
1. Sign in to AWS Console
2. Click on your account name (top-right) → "My Billing Dashboard"
3. In left navigation, click "Billing preferences"
4. Check "Receive Billing Alerts"
5. Click "Save preferences"

**Step 3: Create SNS Topic for Notifications**
1. Open SNS service in AWS Console
2. Click "Topics" in left navigation
3. Click "Create topic"
4. Select "Standard" type
5. Name: "billing-alerts"
6. Click "Create topic"
7. Click "Create subscription"
8. Protocol: "Email"
9. Endpoint: your-email@example.com
10. Click "Create subscription"
11. Check email and confirm subscription

**Step 4: Create Billing Alarms in CloudWatch**
1. Open CloudWatch service
2. In left navigation, click "Alarms" → "All alarms"
3. Click "Create alarm"

**For $10 Alert:**
1. Click "Select metric"
2. Choose "Billing" → "Total Estimated Charge"
3. Select "USD" currency
4. Click "Select metric"
5. Conditions:
   - Threshold type: Static
   - Whenever EstimatedCharges is: Greater than
   - Threshold value: 10
6. Click "Next"
7. Select "In alarm" state
8. Select SNS topic: "billing-alerts"
9. Click "Next"
10. Alarm name: "BillingAlert-10USD"
11. Description: "Alert when charges exceed $10"
12. Click "Next" → "Create alarm"

**Repeat for $50 and $100 alerts** with respective threshold values.

**Verification:**
- Go to CloudWatch → Alarms to see all three alarms
- Check SNS subscription confirmation in email
- Alarms should show "Insufficient data" initially (normal)

### Task 2: IAM User Creation
**Create an IAM user with programmatic access and attach the PowerUserAccess policy.**

**Console-Based Solution:**

**Step 1: Navigate to IAM Service**
1. Sign in to AWS Console
2. In services search box, type "IAM"
3. Click "IAM" from the results

**Step 2: Create New User**
1. In IAM dashboard, click "Users" in left navigation
2. Click "Add users" button
3. User name: "devops-user"
4. Select AWS credential type:
   - ✅ Access key - Programmatic access
   - ❌ Password - AWS Management Console access (uncheck for now)
5. Click "Next: Permissions"

**Step 3: Set Permissions**
1. Select "Attach existing policies directly"
2. In search box, type "PowerUserAccess"
3. Check the box next to "PowerUserAccess" policy
4. Policy description: Provides full access to AWS services and resources, but does not allow management of Users and groups
5. Click "Next: Tags"

**Step 4: Add Tags (Optional)**
1. Click "Add tag"
2. Key: "Department", Value: "DevOps"
3. Key: "Purpose", Value: "Development"
4. Click "Next: Review"

**Step 5: Review and Create**
1. Review user details:
   - User name: devops-user
   - AWS access type: Programmatic access
   - Permissions: PowerUserAccess
2. Click "Create user"

**Step 6: Download Credentials**
1. **IMPORTANT**: Download the CSV file or copy the credentials
2. Access key ID: (save this)
3. Secret access key: (save this - won't be shown again)
4. Click "Download .csv" for backup
5. Click "Close"

**Step 7: Verification**
1. User should now appear in Users list
2. Click on the user name "devops-user"
3. Verify:
   - Access keys section shows 1 access key
   - Permissions tab shows PowerUserAccess policy
   - Groups tab shows "Not in any groups"

**Security Best Practices:**
- Store credentials securely (use AWS CLI configure or environment variables)
- Never commit credentials to code repositories
- Rotate access keys regularly
- Use IAM roles when possible instead of access keys

**Next Steps:**
- Configure AWS CLI: `aws configure`
- Test access: Try creating an S3 bucket or EC2 instance

### Task 3: S3 Bucket Management
**Create an S3 bucket with versioning enabled and upload a file with different versions.**

**Console-Based Solution:**

**Step 1: Navigate to S3 Service**
1. In AWS Console, search for "S3"
2. Click "S3" from services list

**Step 2: Create S3 Bucket**
1. Click "Create bucket" button
2. Bucket settings:
   - Bucket name: "my-versioned-bucket-[random-number]" (must be globally unique)
   - AWS Region: Choose your preferred region (e.g., US East N. Virginia)
3. Object Ownership:
   - Select "ACLs disabled (recommended)"
4. Block Public Access settings:
   - Keep "Block all public access" checked (recommended)
5. Bucket Versioning:
   - ✅ Enable versioning
6. Default encryption:
   - Select "Server-side encryption with Amazon S3 managed keys (SSE-S3)"
7. Click "Create bucket"

**Step 3: Upload First Version of File**
1. Click on your newly created bucket name
2. Click "Upload" button
3. Click "Add files"
4. Create a test file on your computer:
   - Create "test-file.txt" with content: "Version 1 content"
5. Select the file and click "Open"
6. Scroll down and click "Upload"
7. Wait for upload to complete, then click "Close"

**Step 4: Upload Second Version (Same Filename)**
1. In the bucket, click "Upload" again
2. Create an updated version of the same file:
   - Edit "test-file.txt" with content: "Version 2 content - updated"
3. Click "Add files" and select the updated file
4. Click "Upload"
5. Notice: S3 automatically creates a new version

**Step 5: View Object Versions**
1. In the bucket, find your "test-file.txt"
2. Click "Show versions" toggle (top right of objects list)
3. You should see:
   - Latest version with "Version ID"
   - Previous version with different "Version ID"
   - Both versions have the same key name but different version IDs

**Step 6: Manage Versions**
1. Click on each version to see details
2. Download different versions:
   - Click on a version → "Download"
   - Verify content is different for each version
3. To delete a specific version:
   - Check the box next to a specific version
   - Click "Delete" → Confirm

**Step 7: View Version History**
1. Click on the object name (without version)
2. Go to "Versions" tab
3. See complete version history with:
   - Version IDs
   - Upload timestamps
   - Size information
   - Storage class

**Verification Steps:**
- Confirm bucket has versioning enabled in Properties tab
- Verify multiple versions exist for the same object
- Test downloading different versions
- Check that latest version is served by default

**Best Practices:**
- Use versioning for critical data protection
- Set up lifecycle policies to manage old versions
- Monitor storage costs as versions accumulate
- Consider MFA delete for additional protection

### Task 4: EC2 Instance Launch
**Launch a t3.micro EC2 instance in the default VPC with a custom security group allowing SSH and HTTP.**

**Console-Based Solution:**

**Step 1: Navigate to EC2 Service**
1. In AWS Console, search for "EC2"
2. Click "EC2" from services list
3. Ensure you're in the correct region (top-right corner)

**Step 2: Create Security Group First**
1. In EC2 dashboard, click "Security Groups" (left navigation under Network & Security)
2. Click "Create security group"
3. Security group settings:
   - Security group name: "web-server-sg"
   - Description: "Security group for web server allowing SSH and HTTP"
   - VPC: Leave default VPC selected
4. Inbound rules:
   - Click "Add rule"
   - Rule 1: SSH
     - Type: SSH
     - Protocol: TCP
     - Port range: 22
     - Source: My IP (recommended) or 0.0.0.0/0 (less secure)
   - Click "Add rule"
   - Rule 2: HTTP
     - Type: HTTP
     - Protocol: TCP
     - Port range: 80
     - Source: 0.0.0.0/0 (allow from anywhere)
5. Outbound rules: Leave default (all traffic allowed)
6. Click "Create security group"

**Step 3: Create Key Pair (if you don't have one)**
1. In EC2 dashboard, click "Key Pairs" (left navigation under Network & Security)
2. Click "Create key pair"
3. Key pair settings:
   - Name: "my-web-server-key"
   - Key pair type: RSA
   - Private key file format: .pem (for Linux/macOS) or .ppk (for PuTTY)
4. Click "Create key pair"
5. **IMPORTANT**: Download and save the private key file securely

**Step 4: Launch EC2 Instance**
1. In EC2 dashboard, click "Instances" (left navigation)
2. Click "Launch instances"
3. Instance configuration:

**Name and Tags:**
- Name: "WebServer"

**Application and OS Images (Amazon Machine Image):**
- Quick Start: Amazon Linux
- Select "Amazon Linux 2023 AMI" (free tier eligible)
- Architecture: 64-bit (x86)

**Instance Type:**
- Instance type: t3.micro (or t2.micro if t3.micro not available)
- Ensure "Free tier eligible" is shown

**Key Pair:**
- Select existing key pair: "my-web-server-key"
- Acknowledge checkbox about private key access

**Network Settings:**
- Click "Edit"
- VPC: Default VPC
- Subnet: Any availability zone (or select specific one)
- Auto-assign public IP: Enable
- Firewall (security groups): Select existing security group
- Select "web-server-sg" created earlier

**Configure Storage:**
- Root volume: 8 GiB gp3 (default, free tier eligible)
- Encrypted: Leave unchecked for this exercise

**Advanced Details (Optional):**
- Leave most settings as default
- For learning, you can add User data script:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2 Instance</h1>" > /var/www/html/index.html
```

4. Summary:
   - Number of instances: 1
   - Review all settings
5. Click "Launch instance"

**Step 5: Verify Instance Launch**
1. Click "View all instances"
2. Wait for instance state to change from "Pending" to "Running"
3. Instance checks should show "2/2 checks passed"
4. Note the public IP address assigned

**Step 6: Test Connectivity**
1. **SSH Test**: Use terminal or SSH client
   - Command: `ssh -i my-web-server-key.pem ec2-user@[PUBLIC-IP]`
   - Accept fingerprint when prompted
2. **HTTP Test**: Open browser
   - Navigate to: `http://[PUBLIC-IP]`
   - Should see "Hello from EC2 Instance" page (if user data script was used)

**Step 7: Verification Checklist**
- ✅ Instance is in "Running" state
- ✅ Security group allows SSH (port 22) and HTTP (port 80)
- ✅ Instance has public IP address
- ✅ Can SSH to instance successfully
- ✅ HTTP service is accessible (if web server installed)

**Important Notes:**
- Remember to terminate instance when done to avoid charges
- Keep private key file secure and backed up
- Monitor AWS costs in billing dashboard
- Use Elastic IP if you need persistent IP address

### Task 5: Key Pair Management
**Create an EC2 key pair and use it to launch an instance, then connect via SSH.**

**Console-Based Solution:**

**Step 1: Create EC2 Key Pair**
1. In EC2 Console, navigate to "Key Pairs" (left navigation under Network & Security)
2. Click "Create key pair"
3. Key pair configuration:
   - Name: "my-web-server-key"
   - Key pair type: RSA (recommended)
   - Private key file format:
     - .pem (for OpenSSH - Linux, macOS, Windows 10)
     - .ppk (for PuTTY - older Windows versions)
4. Click "Create key pair"
5. **CRITICAL**: Private key file downloads automatically
   - Save it in a secure location (e.g., ~/.ssh/ folder)
   - **This is your only chance to download the private key**

**Step 2: Secure the Private Key (Linux/macOS)**
1. Open terminal
2. Navigate to where you saved the key file
3. Set correct permissions:
   ```bash
   chmod 400 my-web-server-key.pem
   ```
4. Move to SSH directory (optional):
   ```bash
   mv my-web-server-key.pem ~/.ssh/
   ```

**Step 3: Launch Instance with Key Pair**
1. Go to EC2 → Instances → "Launch instances"
2. Configuration:
   - Name: "SSH-Test-Instance"
   - AMI: Amazon Linux 2023 AMI
   - Instance type: t3.micro
   - **Key pair**: Select "my-web-server-key" (the one you just created)
   - Network settings:
     - Create new security group or use existing
     - Allow SSH (port 22) from your IP
3. Click "Launch instance"
4. Wait for instance to reach "Running" state

**Step 4: Get Instance Connection Information**
1. In Instances list, click on your new instance
2. In the details panel, note:
   - Public IPv4 address (e.g., 54.123.45.67)
   - Instance ID
   - Public IPv4 DNS (e.g., ec2-54-123-45-67.compute-1.amazonaws.com)

**Step 5: Connect via SSH**

**Method 1: Using Terminal (Linux/macOS/Windows 10+)**
1. Open terminal/command prompt
2. Navigate to directory containing your private key
3. Connect using:
   ```bash
   ssh -i my-web-server-key.pem ec2-user@[PUBLIC-IP-ADDRESS]
   ```
   Example:
   ```bash
   ssh -i my-web-server-key.pem ec2-user@54.123.45.67
   ```
4. When prompted "Are you sure you want to continue connecting?", type "yes"

**Method 2: Using AWS Console Connect Feature**
1. In EC2 Instances list, select your instance
2. Click "Connect" button
3. Choose "SSH client" tab
4. Follow the provided commands (similar to Method 1)

**Method 3: Using PuTTY (Windows)**
1. Download and install PuTTY
2. Convert .pem to .ppk:
   - Open PuTTYgen
   - Load your .pem file
   - Save as .ppk file
3. Open PuTTY:
   - Host Name: ec2-user@[public-ip]
   - Port: 22
   - Connection → SSH → Auth → Browse for .ppk file
   - Click "Open"

**Step 6: Verify SSH Connection**
1. Once connected, you should see Amazon Linux prompt:
   ```
   [ec2-user@ip-172-31-xx-xx ~]$
   ```
2. Test basic commands:
   ```bash
   whoami          # Should show: ec2-user
   pwd             # Should show: /home/ec2-user
   sudo yum update # Test sudo access
   ```

**Step 7: Additional SSH Features**

**Create SSH Config (Optional)**
1. On your local machine, edit ~/.ssh/config:
   ```
   Host my-aws-server
       HostName [PUBLIC-IP-ADDRESS]
       User ec2-user
       IdentityFile ~/.ssh/my-web-server-key.pem
   ```
2. Now you can connect simply with:
   ```bash
   ssh my-aws-server
   ```

**Troubleshooting Common Issues:**

**Permission Denied Error:**
- Check key file permissions: `ls -la my-web-server-key.pem`
- Should show: `-r-------- 1 user user 1704 date my-web-server-key.pem`
- Fix with: `chmod 400 my-web-server-key.pem`

**Connection Refused:**
- Verify security group allows SSH (port 22) from your IP
- Check instance is in "Running" state
- Verify public IP address is correct

**Wrong Username:**
- Amazon Linux: use `ec2-user`
- Ubuntu: use `ubuntu`
- Red Hat: use `ec2-user` or `root`

**Step 8: Cleanup**
1. When finished, terminate the instance:
   - Select instance → Instance state → Terminate instance
2. Key pair can be reused for future instances
3. To delete key pair: EC2 → Key Pairs → Select → Delete

**Security Best Practices:**
- Never share private key files
- Use different key pairs for different environments
- Regularly rotate key pairs
- Consider using AWS Systems Manager Session Manager for keyless access

### Task 6: Security Group Configuration
**Create a security group that allows inbound traffic on ports 22, 80, and 443 from specific IP ranges.**

**Console-Based Solution:**

**Step 1: Navigate to Security Groups**
1. In EC2 Console, go to "Security Groups" (left navigation under Network & Security)
2. Click "Create security group"

**Step 2: Basic Security Group Configuration**
1. Security group settings:
   - **Security group name**: "multi-port-sg"
   - **Description**: "Security group with multiple ports for web server"
   - **VPC**: Select your default VPC (or specific VPC if needed)

**Step 3: Configure Inbound Rules**
Click "Add rule" for each of the following:

**Rule 1: SSH Access (Restricted IP Range)**
- Type: SSH
- Protocol: TCP (auto-filled)
- Port range: 22 (auto-filled)
- Source: Custom
  - CIDR blocks: "203.0.113.0/24" (example office IP range)
  - OR use "My IP" to automatically use your current IP
- Description: "SSH access from office network"

**Rule 2: HTTP Access (Public)**
- Type: HTTP
- Protocol: TCP (auto-filled)
- Port range: 80 (auto-filled)
- Source: "Anywhere-IPv4" (0.0.0.0/0)
- Description: "HTTP web traffic from anywhere"

**Rule 3: HTTPS Access (Public)**
- Type: HTTPS
- Protocol: TCP (auto-filled)
- Port range: 443 (auto-filled)
- Source: "Anywhere-IPv4" (0.0.0.0/0)
- Description: "HTTPS secure web traffic from anywhere"

**Step 4: Review Inbound Rules Summary**
Your rules should look like:
```
Type      Protocol  Port Range  Source          Description
SSH       TCP       22          203.0.113.0/24  SSH access from office network
HTTP      TCP       80          0.0.0.0/0       HTTP web traffic from anywhere
HTTPS     TCP       443         0.0.0.0/0       HTTPS secure web traffic from anywhere
```

**Step 5: Create Security Group**
1. Review all settings
2. Click "Create security group"
3. Note the Security Group ID (e.g., sg-0123456789abcdef0)

**Step 6: Test with EC2 Instance**

**Launch Test Instance:**
1. Go to EC2 → Instances → "Launch instances"
2. Configuration:
   - Name: "Multi-Port-Test"
   - AMI: Amazon Linux 2023
   - Instance type: t3.micro
   - Key pair: Use existing key pair
   - **Network settings**: Click "Edit"
     - **Security groups**: Select existing security group
     - Choose "multi-port-sg"
3. Launch instance

**Step 7: Verify Security Group Rules**

**Test SSH Access (if from allowed IP range):**
```bash
ssh -i your-key.pem ec2-user@[PUBLIC-IP]
```

**Test HTTP/HTTPS Access:**
1. SSH into instance and setup web server:
   ```bash
   sudo yum update -y
   sudo yum install -y httpd mod_ssl
   sudo systemctl start httpd
   sudo systemctl enable httpd
   
   # Create test page
   echo "<h1>Multi-Port Security Group Test</h1>" | sudo tee /var/www/html/index.html
   ```

2. Test HTTP: `http://[PUBLIC-IP]` in browser
3. For HTTPS testing, you'd need to configure SSL certificates

**Step 8: Advanced Configuration Options**

**Add Custom Port Range:**
1. Edit inbound rules
2. Add rule:
   - Type: Custom TCP Rule
   - Port Range: 8080-8090
   - Source: Specific IP range
   - Description: "Custom application ports"

**Reference Another Security Group:**
1. In Source field, instead of IP range:
2. Select "Custom" → Start typing "sg-" 
3. Choose another security group
4. This allows access from instances in the referenced security group

**Step 9: Security Best Practices**

**IP Range Considerations:**
- Use specific IP ranges instead of 0.0.0.0/0 when possible
- For SSH, always restrict to known IP addresses
- Consider using VPN or bastion hosts for administrative access

**Common IP Range Examples:**
- Single IP: 203.0.113.15/32
- Small network: 203.0.113.0/24 (256 addresses)
- Larger network: 203.0.113.0/16 (65,536 addresses)
- Your current IP: Use "My IP" option

**Step 10: Monitor and Modify**

**View Applied Rules:**
1. Security Groups → Select "multi-port-sg"
2. "Inbound rules" tab shows all current rules
3. "Outbound rules" tab shows egress rules

**Modify Existing Rules:**
1. Select security group → "Inbound rules" → "Edit inbound rules"
2. Modify existing rules or add new ones
3. Click "Save rules"

**Remove Rules:**
1. In edit mode, click "Delete" (X) next to unwanted rules
2. Save changes

**Step 11: Troubleshooting Common Issues**

**Connection Timeouts:**
- Verify correct port is open in security group
- Check Network ACLs (subnet level security)
- Confirm instance is in correct subnet

**SSH Access Denied:**
- Verify your IP is in the allowed CIDR range
- Check if your IP has changed (dynamic IP)
- Confirm SSH service is running on the instance

**Web Access Issues:**
- Ensure web server is running and listening on correct ports
- Check instance health and system logs
- Verify DNS resolution

**Step 12: Cleanup**
1. Terminate test instance when finished
2. Security group can be reused or deleted
3. To delete: Ensure no instances are using it first

**Verification Checklist:**
- ✅ Security group created with descriptive name
- ✅ SSH access restricted to specific IP range
- ✅ HTTP and HTTPS access available from anywhere
- ✅ Rules are properly documented with descriptions
- ✅ Test instance can access appropriate services
- ✅ Security group follows principle of least privilege

### Task 7: Elastic IP
**Allocate an Elastic IP and associate it with your EC2 instance.**

**Console-Based Solution:**

**Step 1: Navigate to Elastic IPs**
1. In EC2 Console, go to "Elastic IPs" (left navigation under Network & Security)
2. Click "Allocate Elastic IP address"

**Step 2: Allocate Elastic IP**
1. Elastic IP settings:
   - **Public IPv4 address pool**: Amazon's pool of IPv4 addresses
   - **Network Border Group**: Use default (your region)
   - **Tags** (optional): 
     - Key: Name, Value: "my-web-server-eip"
     - Key: Purpose, Value: "Static IP for web server"
2. Click "Allocate"

**Step 3: View Allocated Elastic IP**
1. You'll see the allocated IP address (e.g., 54.123.45.67)
2. Note the **Allocation ID** (e.g., eipalloc-1234567890abcdef0)
3. Status shows "Not associated" until attached to an instance

**Step 4: Launch EC2 Instance (if needed)**
If you don't have a running instance:
1. Go to EC2 → Instances → "Launch instances"
2. Configuration:
   - Name: "EIP-Test-Instance"
   - AMI: Amazon Linux 2023
   - Instance type: t3.micro
   - Key pair: Select existing key pair
   - Network settings: Use default
3. Launch and wait for "Running" status

**Step 5: Associate Elastic IP with Instance**
1. Return to "Elastic IPs" page
2. Select your allocated Elastic IP (check the box)
3. Click "Actions" → "Associate Elastic IP address"
4. Association settings:
   - **Resource type**: Instance
   - **Instance**: Select your target instance from dropdown
   - **Private IP address**: Leave default (primary private IP)
   - **Reassociation**: Check this if IP was previously associated
5. Click "Associate"

**Step 6: Verify Association**
1. In Elastic IPs list, check:
   - **Associated instance ID**: Should show your instance
   - **Private IP address**: Shows the private IP
   - **Association ID**: Shows the association ID
2. Go to EC2 Instances and select your instance:
   - **Public IPv4 address**: Should now show the Elastic IP
   - **Public IPv4 DNS**: Updated to reflect the new IP

**Step 7: Test Elastic IP Connectivity**

**Test SSH Connection:**
1. Use the Elastic IP address for SSH:
   ```bash
   ssh -i your-key.pem ec2-user@[ELASTIC-IP-ADDRESS]
   ```
2. Should connect successfully using the static IP

**Test Web Service (Optional):**
1. Install web server on the instance:
   ```bash
   sudo yum update -y
   sudo yum install -y httpd
   sudo systemctl start httpd
   sudo systemctl enable httpd
   echo "<h1>Server with Elastic IP: $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)</h1>" | sudo tee /var/www/html/index.html
   ```
2. Visit `http://[ELASTIC-IP-ADDRESS]` in browser
3. Should display the page with the Elastic IP shown

**Step 8: Benefits of Elastic IP**

**Static IP Address:**
- IP remains the same even if instance is stopped/started
- Useful for DNS records, whitelisting, and external integrations
- Can be moved between instances in the same region

**Test Stop/Start Behavior:**
1. Go to Instances → Select your instance → Instance state → Stop instance
2. Wait for instance to stop
3. Start instance again: Instance state → Start instance
4. Check public IP: Still shows the same Elastic IP
5. Regular instances get new public IPs on restart

**Step 9: Disassociate Elastic IP (if needed)**
1. Go to Elastic IPs → Select your EIP
2. Actions → "Disassociate Elastic IP address"
3. Confirm disassociation
4. Instance will get a new regular public IP (if in public subnet)
5. Elastic IP becomes available for reassignment

**Step 10: Move Elastic IP to Another Instance**
1. Create or identify target instance
2. Go to Elastic IPs → Select your EIP
3. Actions → "Associate Elastic IP address"
4. Select different target instance
5. Check "Reassociation" box (required for already-associated EIPs)
6. Click "Associate"
7. Previous instance loses the Elastic IP immediately

**Step 11: Release Elastic IP**
⚠️ **Important**: You're charged for unused Elastic IPs

**When to Release:**
- No longer needed for any instance
- Want to avoid charges for unassociated IPs

**How to Release:**
1. First disassociate from any instance
2. Go to Elastic IPs → Select your EIP
3. Actions → "Release Elastic IP address"
4. Confirm release
5. **Warning**: Once released, you cannot get the same IP back

**Step 12: Cost Considerations**

**Elastic IP Pricing:**
- **Free** when associated with a running instance
- **Charged** when not associated with an instance
- **Charged** when associated with a stopped instance
- **Charged** for additional IPs beyond the first per instance

**Cost Optimization:**
- Always associate Elastic IPs with running instances
- Release unused Elastic IPs promptly
- Consider using load balancers or DNS for high availability instead

**Step 13: Advanced Use Cases**

**Failover Setup:**
1. Allocate Elastic IP
2. Associate with primary instance
3. For failover: Quickly reassociate to backup instance
4. DNS and external services continue working

**Load Balancer Alternative:**
- For single-instance applications
- Provides static IP without load balancer costs
- Manual failover process

**Step 14: Monitoring and Management**

**CloudWatch Integration:**
- Monitor instance metrics using the Elastic IP
- Set up alarms for instance health
- Track network performance

**Tagging Strategy:**
- Tag Elastic IPs with purpose, environment, owner
- Helps with cost tracking and management
- Use consistent naming conventions

**Step 15: Troubleshooting**

**Association Issues:**
- Verify instance is in VPC (not EC2-Classic)
- Check if instance already has an Elastic IP
- Ensure proper IAM permissions

**Connectivity Issues:**
- Verify security groups allow traffic
- Check Network ACLs
- Confirm routing tables are correct

**Step 16: Cleanup and Best Practices**
1. When finished testing:
   - Disassociate Elastic IP from instance
   - Release the Elastic IP to avoid charges
   - Terminate test instance
2. For production:
   - Keep Elastic IP associated with running instances
   - Document which services depend on the static IP
   - Have backup plans for IP reassignment

**Verification Checklist:**
- ✅ Elastic IP successfully allocated
- ✅ IP associated with EC2 instance
- ✅ Instance accessible via Elastic IP
- ✅ IP remains static after instance stop/start
- ✅ Understanding of cost implications
- ✅ Know how to release IP to avoid charges

### Task 8: S3 Static Website
**Configure an S3 bucket to host a static website with a simple HTML page.**

**Console-Based Solution:**

**Step 1: Create S3 Bucket for Website**
1. Navigate to S3 Console
2. Click "Create bucket"
3. Bucket configuration:
   - **Bucket name**: "my-static-website-[unique-number]" (e.g., my-static-website-2024)
   - **Region**: Choose your preferred region
   - **Object Ownership**: ACLs disabled (recommended)
   - **Block Public Access settings**: 
     - **UNCHECK** "Block all public access" ⚠️
     - **UNCHECK** all four individual settings
     - This is required for public website hosting
   - **Bucket Versioning**: Enable (optional but recommended)
   - **Default encryption**: Server-side encryption with Amazon S3 managed keys
4. **Acknowledge** the warning about public access
5. Click "Create bucket"

**Step 2: Create Website Files**
Create these files on your local computer:

**File 1: index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My AWS Static Website</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>Welcome to My AWS Static Website</h1>
        <nav>
            <a href="index.html">Home</a>
            <a href="about.html">About</a>
        </nav>
    </header>
    <main>
        <section>
            <h2>Hosted on Amazon S3</h2>
            <p>This is a simple static website hosted on Amazon S3. It demonstrates:</p>
            <ul>
                <li>Static website hosting capabilities</li>
                <li>Custom HTML, CSS, and multiple pages</li>
                <li>Global content delivery via S3</li>
            </ul>
        </section>
        <section>
            <h2>Features</h2>
            <div class="feature-grid">
                <div class="feature">
                    <h3>Fast Loading</h3>
                    <p>Served directly from S3 with low latency</p>
                </div>
                <div class="feature">
                    <h3>Cost Effective</h3>
                    <p>Pay only for storage and bandwidth used</p>
                </div>
                <div class="feature">
                    <h3>Highly Available</h3>
                    <p>Built on AWS's reliable infrastructure</p>
                </div>
            </div>
        </section>
    </main>
    <footer>
        <p>&copy; 2024 My Static Website. Powered by AWS S3.</p>
    </footer>
</body>
</html>
```

**File 2: about.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>About - My AWS Static Website</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>About This Website</h1>
        <nav>
            <a href="index.html">Home</a>
            <a href="about.html">About</a>
        </nav>
    </header>
    <main>
        <section>
            <h2>About This Project</h2>
            <p>This static website demonstrates how to:</p>
            <ul>
                <li>Configure S3 bucket for static website hosting</li>
                <li>Upload HTML, CSS, and other static files</li>
                <li>Set up public access permissions</li>
                <li>Access the website via S3 endpoint</li>
            </ul>
        </section>
        <section>
            <h2>Technologies Used</h2>
            <ul>
                <li><strong>Amazon S3:</strong> Static website hosting</li>
                <li><strong>HTML5:</strong> Page structure and content</li>
                <li><strong>CSS3:</strong> Styling and responsive design</li>
            </ul>
        </section>
    </main>
    <footer>
        <p>&copy; 2024 My Static Website. Powered by AWS S3.</p>
    </footer>
</body>
</html>
```

**File 3: styles.css**
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
    color: #333;
    background-color: #f4f4f4;
}

header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 1rem;
    text-align: center;
}

header h1 {
    margin-bottom: 1rem;
}

nav a {
    color: white;
    text-decoration: none;
    margin: 0 1rem;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    transition: background-color 0.3s;
}

nav a:hover {
    background-color: rgba(255, 255, 255, 0.2);
}

main {
    max-width: 1200px;
    margin: 2rem auto;
    padding: 0 1rem;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

section {
    padding: 2rem;
    border-bottom: 1px solid #eee;
}

section:last-child {
    border-bottom: none;
}

h2 {
    color: #667eea;
    margin-bottom: 1rem;
}

h3 {
    color: #764ba2;
    margin-bottom: 0.5rem;
}

.feature-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    margin-top: 1rem;
}

.feature {
    padding: 1.5rem;
    background: #f8f9fa;
    border-radius: 8px;
    border-left: 4px solid #667eea;
}

footer {
    text-align: center;
    padding: 2rem;
    background: #333;
    color: white;
    margin-top: 2rem;
}

ul {
    margin-left: 2rem;
    margin-top: 1rem;
}

li {
    margin-bottom: 0.5rem;
}
```

**File 4: error.html** (Custom error page)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Page Not Found</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>Oops! Page Not Found</h1>
        <nav>
            <a href="index.html">Home</a>
            <a href="about.html">About</a>
        </nav>
    </header>
    <main>
        <section>
            <h2>404 - Page Not Found</h2>
            <p>The page you're looking for doesn't exist.</p>
            <p><a href="index.html">Return to Home</a></p>
        </section>
    </main>
</body>
</html>
```

**Step 3: Upload Files to S3**
1. Go to your bucket in S3 Console
2. Click "Upload"
3. Click "Add files"
4. Select all four files (index.html, about.html, styles.css, error.html)
5. Click "Upload"
6. Wait for upload to complete

**Step 4: Enable Static Website Hosting**
1. In your bucket, go to "Properties" tab
2. Scroll down to "Static website hosting"
3. Click "Edit"
4. Configuration:
   - **Static website hosting**: Enable
   - **Hosting type**: Host a static website
   - **Index document**: `index.html`
   - **Error document**: `error.html` (optional)
5. Click "Save changes"
6. Note the **Bucket website endpoint** (e.g., http://my-static-website-2024.s3-website-us-east-1.amazonaws.com)

**Step 5: Set Bucket Policy for Public Access**
1. Go to "Permissions" tab
2. Scroll to "Bucket policy"
3. Click "Edit"
4. Add this bucket policy (replace `my-static-website-2024` with your bucket name):
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-static-website-2024/*"
        }
    ]
}
```
5. Click "Save changes"

**Step 6: Test Your Website**
1. Copy the **bucket website endpoint** from Properties tab
2. Open the URL in a web browser
3. You should see your homepage with styled content
4. Test navigation between pages
5. Test error page by visiting a non-existent page (e.g., add `/nonexistent.html` to URL)

**Verification Checklist:**
- ✅ S3 bucket created with public access enabled
- ✅ Static website hosting configured with index document
- ✅ HTML, CSS, and error files uploaded successfully
- ✅ Bucket policy allows public read access
- ✅ Website accessible via S3 endpoint URL
- ✅ Navigation between pages works correctly
- ✅ Custom 404 error page displays for invalid URLs

### Task 9: CloudWatch Monitoring
**Set up a CloudWatch alarm to monitor EC2 CPU utilization above 80%.**

**Console-Based Solution:**

**Step 1: Launch or Identify EC2 Instance to Monitor**
If you don't have an instance running:
1. Go to EC2 Console → Instances → "Launch instances"
2. Create a basic instance:
   - Name: "CloudWatch-Test-Instance"
   - AMI: Amazon Linux 2023
   - Instance type: t3.micro
   - Default settings for other options
3. Launch and wait for "Running" status
4. Note the **Instance ID** (e.g., i-1234567890abcdef0)

**Step 2: Create SNS Topic for Notifications**
1. Navigate to **SNS (Simple Notification Service)** Console
2. Click "Topics" in left navigation
3. Click "Create topic"
4. Topic configuration:
   - **Type**: Standard
   - **Name**: "cpu-alerts"
   - **Display name**: "CPU Utilization Alerts"
   - **Description**: "Notifications for high CPU usage"
5. Click "Create topic"
6. Note the **Topic ARN** (e.g., arn:aws:sns:us-east-1:123456789012:cpu-alerts)

**Step 3: Subscribe to SNS Topic**
1. In the newly created topic, click "Create subscription"
2. Subscription configuration:
   - **Protocol**: Email
   - **Endpoint**: your-email@example.com
3. Click "Create subscription"
4. **Important**: Check your email and click "Confirm subscription" in the AWS notification email
5. Subscription status should change to "Confirmed"

**Step 4: Create CloudWatch Alarm**
1. Navigate to **CloudWatch Console**
2. Click "Alarms" in left navigation
3. Click "Create alarm"

**Step 5: Select Metric**
1. Click "Select metric"
2. In metrics browser:
   - Click **"EC2"**
   - Click **"Per-Instance Metrics"**
   - Find your instance ID and select **"CPUUtilization"**
3. Click "Select metric"

**Step 6: Configure Alarm Conditions**
1. Metric configuration (should be pre-filled):
   - **Metric name**: CPUUtilization
   - **Namespace**: AWS/EC2
   - **Dimensions**: InstanceId = [your-instance-id]
2. Conditions:
   - **Threshold type**: Static
   - **Whenever CPUUtilization is**: Greater than threshold
   - **Threshold value**: 80
3. Additional configuration:
   - **Datapoints to alarm**: 2 out of 2
   - **Missing data treatment**: Treat missing data as not breaching threshold
4. Click "Next"

**Step 7: Configure Actions**
1. Notification settings:
   - **Alarm state trigger**: In alarm
   - **Select an SNS topic**: Choose existing topic
   - **SNS topic**: Select "cpu-alerts"
2. Optional additional actions:
   - **EC2 action**: You can add auto scaling or instance recovery
   - **Systems Manager action**: Run automation documents
3. Click "Next"

**Step 8: Add Name and Description**
1. Alarm configuration:
   - **Alarm name**: "HighCPUUtilization"
   - **Alarm description**: "Alarm when CPU exceeds 80% for 2 consecutive periods"
2. Preview your alarm settings
3. Click "Next"

**Step 9: Review and Create**
1. Review all settings:
   - Metric: EC2 CPUUtilization for your instance
   - Condition: > 80% for 2 datapoints
   - Action: Send notification to cpu-alerts topic
2. Click "Create alarm"

**Step 10: Verify Alarm Setup**
1. In CloudWatch Alarms list, your alarm should show:
   - **State**: OK (if CPU is currently below 80%)
   - **Reason**: Threshold Crossed
2. Click on alarm name to see detailed view with graph

**Step 11: Test the Alarm (Simulate High CPU)**
⚠️ **Warning**: This will consume CPU resources on your instance

**Method 1: SSH and Generate CPU Load**
1. SSH into your instance:
   ```bash
   ssh -i your-key.pem ec2-user@[instance-public-ip]
   ```
2. Install stress testing tool:
   ```bash
   sudo yum update -y
   sudo yum install -y stress
   ```
3. Generate CPU load for 10 minutes:
   ```bash
   stress --cpu 1 --timeout 600s
   ```
4. Monitor in CloudWatch - should trigger alarm after ~10 minutes

**Method 2: Using Python Script**
1. SSH into instance and create CPU load script:
   ```bash
   cat > cpu_load.py << 'EOF'
   import time
   import threading
   
   def cpu_load():
       while True:
           pass
   
   # Start CPU intensive threads
   for i in range(2):  # Adjust based on CPU cores
       thread = threading.Thread(target=cpu_load)
       thread.daemon = True
       thread.start()
   
   # Run for 10 minutes
   time.sleep(600)
   EOF
   
   python3 cpu_load.py
   ```

**Step 12: Monitor Alarm Behavior**
1. In CloudWatch Console → Alarms:
   - Refresh the page periodically
   - Watch alarm state change from "OK" → "In alarm"
2. Check your email for alarm notification
3. View the alarm history:
   - Click alarm name → "History" tab
   - See state changes and reasons

**Step 13: View CloudWatch Metrics Graph**
1. In alarm details, view the metric graph
2. You should see:
   - CPU utilization spike above 80%
   - Red line indicating alarm threshold
   - Time periods when alarm was triggered

**Step 14: Create Additional Useful Alarms**

**Low CPU Alarm (Optional):**
1. Create another alarm with:
   - Condition: CPU < 10% for 30 minutes
   - Action: Send notification (instance might be idle/unused)

**Instance Status Check Alarm:**
1. Select metric: EC2 → Per-Instance Metrics → StatusCheckFailed
2. Condition: > 0 for 2 datapoints
3. Action: Send notification + recover instance

**Memory Utilization Alarm (Requires CloudWatch Agent):**
1. Install CloudWatch agent on instance first
2. Configure custom metrics for memory usage
3. Create alarm for memory > 85%

**Step 15: Alarm Management**

**Edit Existing Alarm:**
1. Select alarm → Actions → "Edit"
2. Modify threshold, period, or actions
3. Save changes

**Disable Alarm Temporarily:**
1. Select alarm → Actions → "Disable actions"
2. Alarm continues monitoring but won't send notifications

**Delete Alarm:**
1. Select alarm → Actions → "Delete"
2. Confirm deletion

**Step 16: Cost Optimization**

**CloudWatch Pricing Considerations:**
- First 10 metrics and alarms are free
- Additional alarms: $0.10 per alarm per month
- Custom metrics: $0.30 per metric per month
- API requests: $0.01 per 1,000 requests

**Best Practices:**
- Use composite alarms for complex conditions
- Set appropriate evaluation periods to avoid false alarms
- Clean up unused alarms regularly

**Step 17: Advanced Features**

**Composite Alarms:**
1. Combine multiple alarms with AND/OR logic
2. Useful for complex monitoring scenarios
3. Create via CloudWatch → Alarms → "Create alarm" → "Composite alarm"

**Anomaly Detection:**
1. CloudWatch can learn normal patterns
2. Alert on unusual behavior without fixed thresholds
3. Useful for dynamic workloads

**Step 18: Integration with Auto Scaling**
1. CloudWatch alarms can trigger Auto Scaling actions
2. Scale out when CPU > 80%
3. Scale in when CPU < 20%
4. Configure via Auto Scaling Groups

**Step 19: Troubleshooting Common Issues**

**Alarm Not Triggering:**
- Verify instance is sending metrics to CloudWatch
- Check evaluation periods and datapoints settings
- Ensure instance has CloudWatch permissions

**No Email Notifications:**
- Confirm SNS subscription is confirmed
- Check spam folder
- Verify email address is correct

**Missing Metrics:**
- Some metrics require CloudWatch agent installation
- Basic EC2 metrics are available by default
- Custom application metrics need explicit publishing

**Step 20: Cleanup**
When finished testing:
1. Delete the CloudWatch alarm:
   - CloudWatch → Alarms → Select → Delete
2. Delete SNS topic and subscription:
   - SNS → Topics → Select → Delete
3. Terminate test EC2 instance to avoid charges
4. Note: CloudWatch retains metric data based on retention periods

**Verification Checklist:**
- ✅ EC2 instance running and sending metrics
- ✅ SNS topic created with confirmed email subscription
- ✅ CloudWatch alarm created with correct threshold (80%)
- ✅ Alarm configured to trigger after 2 consecutive periods
- ✅ Alarm action sends notification to SNS topic
- ✅ Successfully tested alarm by generating high CPU load
- ✅ Received email notification when alarm triggered
- ✅ Alarm state changes visible in CloudWatch console
- ✅ Understanding of CloudWatch pricing and best practices

### Task 10: RDS Database
**Create a MySQL RDS instance in a private subnet with proper security group configuration.**

**Console-Based Solution:**

**Step 1: Plan Your RDS Deployment**
Before creating the database, understand the architecture:
- **Private subnet**: Database not directly accessible from internet
- **Security groups**: Control access to database port (3306 for MySQL)
- **Subnet group**: Defines which subnets RDS can use
- **Multi-AZ**: High availability across availability zones

**Step 2: Create DB Subnet Group**
1. Navigate to **RDS Console**
2. Click "Subnet groups" in left navigation
3. Click "Create DB subnet group"
4. Configuration:
   - **Name**: "private-subnet-group"
   - **Description**: "Subnet group for private RDS instances"
   - **VPC**: Select your default VPC (or specific VPC)
5. **Add subnets**:
   - **Availability Zones**: Select at least 2 AZs (required for Multi-AZ)
   - **Subnets**: Choose private subnets in each AZ
   - If you only have public subnets, select 2 different ones
6. Click "Create"

**Step 3: Create Security Group for RDS**
1. Go to **EC2 Console** → Security Groups
2. Click "Create security group"
3. Configuration:
   - **Security group name**: "rds-mysql-sg"
   - **Description**: "Security group for MySQL RDS database"
   - **VPC**: Same VPC as your subnet group

**Step 4: Configure RDS Security Group Rules**

**Inbound Rules:**
1. Click "Add rule"
2. **Rule 1: MySQL/Aurora access from application servers**
   - Type: MySQL/Aurora (port 3306 auto-filled)
   - Protocol: TCP
   - Port range: 3306
   - Source: Custom
     - Option A: Another security group (sg-app-servers)
     - Option B: Specific CIDR (e.g., 10.0.0.0/16 for VPC internal)
     - Option C: "My IP" for testing from your computer
   - Description: "MySQL access from application tier"

3. **Rule 2: Management access (optional)**
   - Type: MySQL/Aurora
   - Source: Your IP or bastion host security group
   - Description: "Database administration access"

4. Click "Create security group"

**Step 5: Create RDS MySQL Instance**
1. In **RDS Console**, click "Databases"
2. Click "Create database"

**Step 6: Choose Database Creation Method**
1. **Database creation method**: Standard create
2. **Engine options**:
   - Engine type: MySQL
   - Edition: MySQL Community
   - Version: MySQL 8.0.35 (or latest stable)

**Step 7: Templates and Instance Configuration**
1. **Templates**: Free tier (for learning) or Production (for real use)
2. **DB instance identifier**: "myapp-mysql"
3. **Master username**: "admin" (or your preferred username)
4. **Master password**:
   - Choose "Auto generate a password" OR
   - "Manage master credentials in AWS Secrets Manager" (recommended) OR
   - Set password manually: Use strong password (e.g., MySecurePassword123!)
5. **DB instance class**:
   - **Standard classes**: db.t3.micro (free tier) or db.t3.small
   - For production: db.r5.large or higher

**Step 8: Storage Configuration**
1. **Storage type**: General Purpose SSD (gp2)
2. **Allocated storage**: 20 GB (minimum for MySQL)
3. **Storage autoscaling**:
   - Enable storage autoscaling: ✅ (recommended)
   - Maximum storage threshold: 100 GB
4. **IOPS**: Leave default for gp2

**Step 9: Connectivity Settings**
1. **Compute resource**: Don't connect to an EC2 compute resource
2. **VPC**: Select your VPC
3. **DB subnet group**: Select "private-subnet-group" (created in Step 2)
4. **Public access**: No ⚠️ (This ensures database is private)
5. **VPC security groups**: Choose existing
   - Remove default security group
   - Add "rds-mysql-sg" (created in Step 3)
6. **Availability Zone**: No preference (let AWS choose)
7. **Database port**: 3306 (MySQL default)

**Step 10: Database Authentication**
1. **Database authentication**: Password authentication
2. **IAM database authentication**: Enable (optional, for enhanced security)

**Step 11: Additional Configuration**
1. **Database options**:
   - **Initial database name**: "myapp" (creates a default schema)
   - **Parameter group**: default.mysql8.0
   - **Option group**: default:mysql-8-0
2. **Backup**:
   - **Automated backups**: Enable
   - **Backup retention period**: 7 days
   - **Backup window**: Select preferred time (e.g., 03:00-04:00 UTC)
   - **Copy tags to snapshots**: Enable
3. **Monitoring**:
   - **Performance Insights**: Enable (free for 7 days)
   - **Monitoring role**: Default
   - **Enhanced monitoring**: Enable with 60-second granularity (optional)
4. **Log exports**:
   - **Error log**: ✅
   - **General log**: ✅ (for debugging)
   - **Slow query log**: ✅ (for performance tuning)

**Step 12: Maintenance and Deletion Protection**
1. **Maintenance**:
   - **Auto minor version upgrade**: Enable
   - **Maintenance window**: Select preferred time
2. **Deletion protection**: Enable ⚠️ (prevents accidental deletion)

**Step 13: Encryption (Important for Production)**
1. **Encryption**:
   - **Encrypt this database**: Enable
   - **AWS KMS key**: Default (aws/rds) or create custom key
2. **Performance Insights encryption**: Enable

**Step 14: Create the Database**
1. Review all settings in the summary
2. **Estimated monthly costs**: Review pricing estimate
3. Click "Create database"
4. **Important**: If you chose auto-generate password, immediately:
   - Click "View credential details"
   - Save the password securely
   - Note: You can only see this password once

**Step 15: Monitor Database Creation**
1. Database creation takes 10-20 minutes
2. Status progression: "Creating" → "Backing-up" → "Available"
3. In Databases list, monitor the status
4. Once "Available", note the **Endpoint** (e.g., myapp-mysql.cluster-xyz.us-east-1.rds.amazonaws.com)

**Step 16: Test Database Connectivity**

**Option A: From EC2 Instance in Same VPC**
1. Launch EC2 instance in same VPC
2. SSH into the instance
3. Install MySQL client:
   ```bash
   # Amazon Linux 2023
   sudo yum update -y
   sudo yum install -y mysql
   
   # Ubuntu
   sudo apt update
   sudo apt install -y mysql-client
   ```
4. Connect to RDS:
   ```bash
   mysql -h myapp-mysql.cluster-xyz.us-east-1.rds.amazonaws.com -u admin -p
   ```
5. Enter password when prompted
6. Test connection:
   ```sql
   SHOW DATABASES;
   USE myapp;
   CREATE TABLE test_table (id INT, name VARCHAR(50));
   INSERT INTO test_table VALUES (1, 'Test Record');
   SELECT * FROM test_table;
   ```

**Option B: Using MySQL Workbench (if security group allows)**
1. Download MySQL Workbench on your computer
2. Create new connection:
   - Connection Method: Standard (TCP/IP)
   - Hostname: [RDS endpoint]
   - Port: 3306
   - Username: admin
   - Password: [your password]
3. Test connection

**Step 17: Create Application Database and User**
Once connected, create application-specific database and user:
```sql
-- Create application database
CREATE DATABASE webapp_production;

-- Create application user with limited privileges
CREATE USER 'webapp_user'@'%' IDENTIFIED BY 'WebAppPassword123!';

-- Grant privileges only to the application database
GRANT SELECT, INSERT, UPDATE, DELETE ON webapp_production.* TO 'webapp_user'@'%';

-- Flush privileges
FLUSH PRIVILEGES;

-- Verify user creation
SELECT User, Host FROM mysql.user WHERE User = 'webapp_user';
```

**Step 18: Configure Application Connection**
Example connection strings for common frameworks:

**PHP (PDO):**
```php
$dsn = "mysql:host=myapp-mysql.cluster-xyz.us-east-1.rds.amazonaws.com;dbname=webapp_production;charset=utf8mb4";
$pdo = new PDO($dsn, 'webapp_user', 'WebAppPassword123!');
```

**Python (SQLAlchemy):**
```python
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://webapp_user:WebAppPassword123!@myapp-mysql.cluster-xyz.us-east-1.rds.amazonaws.com/webapp_production')
```

**Node.js (MySQL2):**
```javascript
const mysql = require('mysql2');
const connection = mysql.createConnection({
  host: 'myapp-mysql.cluster-xyz.us-east-1.rds.amazonaws.com',
  user: 'webapp_user',
  password: 'WebAppPassword123!',
  database: 'webapp_production'
});
```

**Step 19: Database Maintenance Tasks**

**Create Manual Snapshot:**
1. RDS Console → Databases → Select your database
2. Actions → "Take snapshot"
3. Snapshot identifier: "myapp-mysql-manual-snapshot-2025-10-22"
4. Click "Take snapshot"

**Modify Database (if needed):**
1. Select database → "Modify"
2. Can change: instance class, storage, security groups, etc.
3. **Apply immediately** or **During maintenance window**

**Monitor Performance:**
1. Database → Monitoring tab
2. View CPU, connections, read/write IOPS
3. Performance Insights for detailed query analysis

**Step 20: Security Best Practices**

**Network Security:**
- ✅ Database in private subnets
- ✅ Security groups restrict access to port 3306
- ✅ No public accessibility
- ✅ Application servers in same VPC

**Access Control:**
- ✅ Strong master password
- ✅ Application-specific database users
- ✅ Principle of least privilege for database permissions
- ✅ Consider IAM database authentication

**Encryption:**
- ✅ Encryption at rest enabled
- ✅ Encryption in transit (SSL/TLS connections)
- ✅ Encrypted automated backups

**Step 21: Cost Optimization**

**Instance Sizing:**
- Start with smaller instances (db.t3.micro for dev)
- Scale up based on actual usage patterns
- Use Performance Insights to identify bottlenecks

**Storage:**
- Monitor storage usage and growth
- Use storage autoscaling to avoid manual intervention
- Consider Magnetic storage for dev/test environments

**Reserved Instances:**
- For production workloads, consider 1-year or 3-year reserved instances
- Can save 20-60% compared to on-demand pricing

**Step 22: Troubleshooting Common Issues**

**Cannot Connect to Database:**
- Verify security group allows connections on port 3306
- Check if database is in "Available" state
- Ensure client is in same VPC or has network connectivity
- Verify endpoint URL and port

**Performance Issues:**
- Check CPU and memory utilization in CloudWatch
- Use Performance Insights to identify slow queries
- Consider read replicas for read-heavy workloads
- Optimize database schemas and indexes

**Step 23: Cleanup**
When finished with testing:
1. **Delete Read Replicas** (if any) first
2. **Disable deletion protection**:
   - Select database → Modify → Uncheck deletion protection
3. **Delete database**:
   - Select database → Actions → Delete
   - Choose whether to create final snapshot
   - Type "delete me" to confirm
4. **Delete subnet group**:
   - RDS → Subnet groups → Select → Delete
5. **Delete security group**:
   - EC2 → Security Groups → Select → Delete (after database is deleted)

**Verification Checklist:**
- ✅ DB subnet group created with multiple AZs
- ✅ Security group configured for MySQL access
- ✅ RDS MySQL instance created in private subnet
- ✅ Database is not publicly accessible
- ✅ Encryption enabled for data at rest
- ✅ Automated backups configured
- ✅ Database connection tested from EC2 instance
- ✅ Application database and user created
- ✅ Understanding of monitoring and maintenance procedures
- ✅ Security best practices implemented

### Task 11: Route 53 Basics
**Register a domain (or use existing) and create a hosted zone with basic DNS records.**

**Console-Based Solution:**

**Step 1: Understanding Route 53 Components**
- **Hosted Zone**: Container for DNS records for a domain
- **DNS Records**: Instructions on how to respond to DNS queries
- **Domain Registration**: Optional service to register new domains
- **Name Servers**: AWS-provided name servers for your domain

**Step 2: Option A - Register New Domain with Route 53**

**If you want to register a new domain:**
1. Navigate to **Route 53 Console**
2. Click "Registered domains" in left navigation
3. Click "Register domain"
4. **Search for domain**:
   - Enter desired domain name (e.g., "myawsproject2025")
   - Click "Search"
   - Choose from available TLDs (.com, .net, .org, etc.)
5. **Domain configuration**:
   - Duration: 1 year (minimum)
   - Auto-renewal: Enable (recommended)
6. **Contact information**:
   - Fill in registrant, admin, and tech contact details
   - **Privacy protection**: Enable (recommended)
7. **Terms and conditions**: Accept
8. **Payment**: Add payment method and complete registration
9. **Note**: Domain registration takes a few minutes to process

**Step 2: Option B - Use Existing Domain (External Registrar)**

**If you have a domain registered elsewhere:**
1. You'll create a hosted zone in Route 53
2. Update your domain's name servers at your registrar
3. This allows Route 53 to handle DNS for your domain

**For this tutorial, we'll proceed with Option B using a test domain**

**Step 3: Create Hosted Zone**
1. In **Route 53 Console**, click "Hosted zones"
2. Click "Create hosted zone"
3. **Hosted zone configuration**:
   - **Domain name**: "example.com" (use your actual domain)
   - **Description**: "DNS zone for my website and applications"
   - **Type**: Public hosted zone
   - **Tags** (optional):
     - Key: Environment, Value: Production
     - Key: Project, Value: MyWebsite
4. Click "Create hosted zone"

**Step 4: Note the Name Servers**
1. In your new hosted zone, you'll see default records:
   - **NS record**: Lists 4 AWS name servers
   - **SOA record**: Start of Authority record
2. **Important**: Copy the 4 name servers (e.g.):
   - ns-123.awsdns-12.com
   - ns-456.awsdns-45.net
   - ns-789.awsdns-78.org
   - ns-012.awsdns-01.co.uk
3. These need to be configured at your domain registrar

**Step 5: Create A Record (Points Domain to IP Address)**
1. In your hosted zone, click "Create record"
2. **Record configuration**:
   - **Record name**: Leave blank (for root domain) or enter "www"
   - **Record type**: A – Routes traffic to an IPv4 address
   - **Value**: Enter an IP address (e.g., 203.0.113.1)
     - Use your web server's public IP
     - Or use a test IP for now
   - **TTL (Time to Live)**: 300 seconds (5 minutes)
   - **Routing policy**: Simple routing
3. Click "Create records"

**Step 6: Create CNAME Record (Alias for Another Domain)**
1. Click "Create record" again
2. **Record configuration**:
   - **Record name**: "blog" (creates blog.example.com)
   - **Record type**: CNAME – Routes traffic to another domain name
   - **Value**: "www.example.com"
   - **TTL**: 300 seconds
3. Click "Create records"

**Step 7: Create Additional Useful Records**

**MX Record (for Email):**
1. Create record:
   - **Record name**: Leave blank (root domain)
   - **Record type**: MX – Routes traffic to mail servers
   - **Value**: "10 mail.example.com" (priority 10, mail server)
   - **TTL**: 300
2. Create records

**TXT Record (for Domain Verification):**
1. Create record:
   - **Record name**: Leave blank or "_verification"
   - **Record type**: TXT – Holds text information
   - **Value**: "v=spf1 include:_spf.google.com ~all" (SPF record example)
   - **TTL**: 300
2. Create records

**AAAA Record (IPv6):**
1. Create record:
   - **Record name**: "www"
   - **Record type**: AAAA – Routes traffic to an IPv6 address
   - **Value**: "2001:db8::1" (example IPv6 address)
   - **TTL**: 300
2. Create records

**Step 8: Create Alias Record (Route 53 Special Feature)**

**Alias to S3 Static Website:**
1. Create record:
   - **Record name**: Leave blank or "www"
   - **Record type**: A
   - **Alias**: ✅ Enable
   - **Route traffic to**: 
     - "Alias to S3 website endpoint"
     - Region: Your S3 bucket region
     - S3 bucket: Your static website bucket
   - **Routing policy**: Simple routing
2. Create records

**Alias to CloudFront Distribution:**
1. Create record:
   - **Record name**: "cdn"
   - **Record type**: A
   - **Alias**: ✅ Enable
   - **Route traffic to**: 
     - "Alias to CloudFront distribution"
     - Distribution: Select your CloudFront distribution
2. Create records

**Step 9: Verify DNS Records**
1. In hosted zone, review all created records:
   ```
   Name                Type    Value
   example.com         A       203.0.113.1
   www.example.com     A       203.0.113.1
   blog.example.com    CNAME   www.example.com
   example.com         MX      10 mail.example.com
   example.com         TXT     "v=spf1 include:_spf.google.com ~all"
   www.example.com     AAAA    2001:db8::1
   ```

**Step 10: Test DNS Resolution**

**Using Online DNS Tools:**
1. Visit tools like:
   - whatsmydns.net
   - dnschecker.org
   - nslookup.io
2. Enter your domain and check different record types
3. **Note**: Changes may take up to 48 hours to propagate globally

**Using Command Line Tools:**
1. **Windows PowerShell:**
   ```powershell
   nslookup www.example.com
   nslookup -type=MX example.com
   nslookup -type=TXT example.com
   ```

2. **Linux/macOS:**
   ```bash
   dig www.example.com
   dig MX example.com
   dig TXT example.com
   ```

**Step 11: Advanced Routing Policies**

**Weighted Routing (Traffic Distribution):**
1. Create record with 70% traffic:
   - Record name: "api"
   - Type: A
   - Value: 203.0.113.1
   - TTL: 60
   - Routing policy: Weighted
   - Weight: 70
   - Set ID: "server-1"

2. Create record with 30% traffic:
   - Record name: "api"
   - Type: A
   - Value: 203.0.113.2
   - TTL: 60
   - Routing policy: Weighted
   - Weight: 30
   - Set ID: "server-2"

**Latency-Based Routing:**
1. Create record for US East:
   - Record name: "app"
   - Type: A
   - Value: 203.0.113.1
   - Routing policy: Latency
   - Region: us-east-1
   - Set ID: "us-east"

2. Create record for EU West:
   - Record name: "app"
   - Type: A  
   - Value: 203.0.113.2
   - Routing policy: Latency
   - Region: eu-west-1
   - Set ID: "eu-west"

**Step 12: Health Checks and Failover**

**Create Health Check:**
1. Route 53 → Health checks → "Create health check"
2. Configuration:
   - **Name**: "webserver-health"
   - **Type**: HTTP
   - **IP address**: 203.0.113.1
   - **Port**: 80
   - **Path**: /health (or /)
   - **Request interval**: 30 seconds
   - **Failure threshold**: 3
3. Create health check

**Create Failover Routing:**
1. Primary record:
   - Record name: "www"
   - Type: A
   - Value: 203.0.113.1
   - Routing policy: Failover
   - Failover record type: Primary
   - Health check: Select created health check
   - Set ID: "primary"

2. Secondary record:
   - Record name: "www"
   - Type: A
   - Value: 203.0.113.2
   - Routing policy: Failover
   - Failover record type: Secondary
   - Set ID: "secondary"

**Step 13: Monitoring and Logging**

**CloudWatch Integration:**
1. Route 53 automatically sends metrics to CloudWatch
2. Monitor:
   - Query count
   - Health check status
   - Resolver query logs

**Query Logging:**
1. In hosted zone → "Configure query logging"
2. Create CloudWatch Logs log group
3. Enable query logging for detailed DNS query analysis

**Step 14: Domain Transfer (If Using External Domain)**

**Update Name Servers at Registrar:**
1. Log into your domain registrar (GoDaddy, Namecheap, etc.)
2. Find DNS/Name Server settings
3. Replace default name servers with Route 53 name servers
4. Save changes
5. **Wait 24-48 hours** for global DNS propagation

**Verify Transfer:**
1. Use `nslookup` or `dig` to confirm Route 53 is authoritative
2. Check that your DNS records resolve correctly

**Step 15: Cost Optimization**

**Route 53 Pricing:**
- **Hosted zone**: $0.50 per month per hosted zone
- **DNS queries**: $0.40 per million queries
- **Health checks**: $0.50 per month per health check
- **Domain registration**: Varies by TLD

**Best Practices:**
- Use appropriate TTL values (longer for stable records)
- Monitor query patterns and optimize record structure
- Use alias records when possible (no charge for queries)

**Step 16: Security Best Practices**

**DNSSEC (Domain Name System Security Extensions):**
1. In hosted zone → "DNSSEC signing"
2. Enable DNSSEC to prevent DNS spoofing
3. **Note**: Requires coordination with domain registrar

**Access Control:**
- Use IAM policies to control who can modify DNS records
- Enable CloudTrail for DNS change auditing
- Consider using separate AWS accounts for critical domains

**Step 17: Troubleshooting Common Issues**

**DNS Not Resolving:**
- Verify name servers are correctly set at registrar
- Check DNS propagation using online tools
- Confirm TTL values aren't too high during testing

**Health Check Failures:**
- Verify target server is responding on specified port
- Check security groups allow health check traffic
- Confirm health check path returns 200 status

**Slow DNS Resolution:**
- Consider lower TTL values for dynamic records
- Use geographically distributed health checks
- Implement proper caching strategies

**Step 18: Integration with Other AWS Services**

**S3 Static Website:**
- Use alias records pointing to S3 website endpoints
- No charge for alias record queries

**CloudFront CDN:**
- Route apex domain to CloudFront distribution
- Enable global content delivery

**Application Load Balancer:**
- Use alias records for load balancer endpoints
- Automatic health checking and failover

**Step 19: Cleanup**
When finished with testing:
1. **Delete hosted zone**:
   - Remove all non-default records first
   - Hosted zones → Select → Delete
2. **Update domain registrar** (if applicable):
   - Restore original name servers
3. **Cancel health checks** to avoid ongoing charges

**Verification Checklist:**
- ✅ Hosted zone created for domain
- ✅ NS and SOA records automatically created
- ✅ A record points domain to IP address
- ✅ CNAME record creates subdomain alias
- ✅ MX record configured for email routing
- ✅ TXT record added for domain verification
- ✅ DNS resolution tested using online tools
- ✅ Understanding of different routing policies
- ✅ Health checks configured for failover scenarios
- ✅ Cost implications understood

### Task 12: Lambda Function
**Create a simple Lambda function that logs "Hello World" and test it.**

**Console-Based Solution:**

**Step 1: Navigate to Lambda Console**
1. Go to **AWS Lambda Console**
2. Click "Functions" in left navigation
3. Click "Create function"

**Step 2: Choose Function Creation Method**
1. **Function creation method**: Author from scratch
2. **Basic information**:
   - **Function name**: "hello-world-function"
   - **Runtime**: Python 3.12 (or latest available)
   - **Architecture**: x86_64

**Step 3: Configure Execution Role**
1. **Permissions**: Expand this section
2. **Execution role**: Create a new role with basic Lambda permissions
3. **Role name**: "hello-world-function-role-[random-string]" (auto-generated)
4. This role will have basic CloudWatch logging permissions

**Step 4: Create the Function**
1. Click "Create function"
2. Wait for function creation to complete
3. You'll be redirected to the function configuration page

**Step 5: Write the Function Code**
1. In the **Code** tab, you'll see a default code editor
2. Replace the default code with:

```python
import json
import logging

# Configure logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    """
    AWS Lambda function handler
    
    Args:
        event: Input event data
        context: Lambda runtime information
        
    Returns:
        dict: Response with statusCode and body
    """
    
    # Log the Hello World message
    logger.info("Hello World - Lambda function executed!")
    
    # Log the incoming event for debugging
    logger.info(f"Received event: {json.dumps(event)}")
    
    # Create response
    response = {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps({
            'message': 'Hello World from Lambda!',
            'timestamp': context.aws_request_id,
            'function_name': context.function_name,
            'function_version': context.function_version,
            'remaining_time_ms': context.get_remaining_time_in_millis(),
            'event_received': event
        }, indent=2)
    }
    
    logger.info(f"Returning response: {response}")
    return response
```

3. Click "Deploy" to save the code changes

**Step 6: Configure Function Settings (Optional)**

**Basic Settings:**
1. Scroll down to "Configuration" tab
2. Click "General configuration" → "Edit"
3. **Timeout**: 3 minutes (default is usually fine)
4. **Memory**: 128 MB (sufficient for this simple function)
5. **Description**: "Simple Hello World Lambda function for testing"
6. Click "Save"

**Environment Variables (Optional):**
1. Go to "Configuration" → "Environment variables" → "Edit"
2. Add variables:
   - Key: `LOG_LEVEL`, Value: `INFO`
   - Key: `ENVIRONMENT`, Value: `development`
3. Click "Save"

**Step 7: Test the Function**

**Create Test Event:**
1. In function overview, click "Test" button
2. **Event template**: hello-world (or create new)
3. **Event name**: "test-event"
4. **Event JSON**:
```json
{
  "key1": "value1",
  "key2": "value2",
  "name": "AWS Lambda Test",
  "message": "Testing Hello World function"
}
```
5. Click "Save"

**Execute Test:**
1. Click "Test" button to run the function
2. View execution results in the response section
3. **Execution result**: Should show "succeeded"
4. **Response**: Should show your function's return value
5. **Function logs**: Click "Logs" to see CloudWatch logs

**Step 8: View Function Logs**
1. **Method 1: In Lambda Console**
   - Scroll down to see "Log output" section
   - Shows recent log entries from CloudWatch

2. **Method 2: CloudWatch Logs**
   - Go to CloudWatch Console → Log groups
   - Find log group: `/aws/lambda/hello-world-function`
   - Click to view detailed logs with timestamps

**Step 9: Monitor Function Performance**
1. In Lambda function → "Monitoring" tab
2. View metrics:
   - **Invocations**: Number of times function was called
   - **Duration**: Average execution time
   - **Errors**: Failed executions
   - **Throttles**: Rate-limited executions

**Step 10: Create Different Test Events**

**Empty Event Test:**
```json
{}
```

**API Gateway Event Test:**
1. Choose event template: "API Gateway AWS Proxy"
2. Simulates HTTP request from API Gateway
3. Test how function handles different event types

**CloudWatch Event Test:**
```json
{
  "source": "aws.cloudwatch",
  "detail-type": "Scheduled Event",
  "detail": {}
}
```

**Step 11: Set Up Function URL (Optional)**
1. In function configuration → "Function URL"
2. Click "Create function URL"
3. **Auth type**: NONE (for testing - use IAM for production)
4. **CORS settings**: Configure if needed
5. Click "Save"
6. Copy the Function URL
7. Test by visiting URL in browser or using curl:
   ```bash
   curl -X POST https://your-function-url/ \
     -H "Content-Type: application/json" \
     -d '{"test": "data"}'
   ```

**Step 12: Add Error Handling**
Update your function code to include error handling:

```python
import json
import logging
import traceback

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        logger.info("Hello World - Lambda function executed!")
        logger.info(f"Received event: {json.dumps(event)}")
        
        # Simulate some processing
        if event.get('error_test'):
            raise ValueError("Simulated error for testing")
        
        response = {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json'
            },
            'body': json.dumps({
                'message': 'Hello World from Lambda!',
                'success': True,
                'timestamp': context.aws_request_id,
                'event_data': event
            })
        }
        
        return response
        
    except Exception as e:
        logger.error(f"Error processing request: {str(e)}")
        logger.error(traceback.format_exc())
        
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json'
            },
            'body': json.dumps({
                'message': 'Internal server error',
                'error': str(e),
                'success': False
            })
        }
```

**Step 13: Version and Alias Management**

**Create Function Version:**
1. Actions → "Publish new version"
2. **Version description**: "Initial Hello World implementation"
3. Click "Publish"
4. Note the version number (e.g., 1)

**Create Alias:**
1. Actions → "Create alias"
2. **Name**: "PROD"
3. **Version**: 1
4. **Description**: "Production version of Hello World function"
5. Click "Save"

**Step 14: Add Trigger (Optional)**

**CloudWatch Events Trigger:**
1. Function overview → "Add trigger"
2. **Source**: CloudWatch Events/EventBridge
3. **Rule**: Create new rule
   - **Rule name**: "daily-hello"
   - **Rule type**: Schedule expression
   - **Schedule expression**: `rate(1 day)`
4. Click "Add"

**S3 Trigger:**
1. Add trigger → S3
2. **Bucket**: Select existing bucket
3. **Event type**: All object create events
4. **Suffix**: .txt (optional filter)
5. Click "Add"

**Step 15: Function Dependencies (Advanced)**

**Using External Libraries:**
1. Create deployment package with dependencies:
   - Create folder locally: `hello-world-function`
   - Add `lambda_function.py` with your code
   - Install dependencies: `pip install requests -t .`
   - Zip the entire folder
   - Upload via "Upload from" → ".zip file"

**Using Lambda Layers:**
1. Create layer with common libraries
2. Attach layer to your function
3. Reduces deployment package size

**Step 16: Security Best Practices**

**IAM Role Permissions:**
1. Go to IAM → Roles → Find your function's role
2. Review attached policies
3. Add only necessary permissions
4. Example additional policies:
   - S3 read access for processing files
   - DynamoDB read/write for data storage
   - SNS publish for notifications

**VPC Configuration (if needed):**
1. Configuration → VPC
2. Connect function to specific VPC
3. Configure security groups and subnets
4. **Note**: Adds cold start latency

**Step 17: Cost Optimization**

**Lambda Pricing Components:**
- **Requests**: $0.20 per 1M requests
- **Duration**: Based on memory allocation and execution time
- **Free tier**: 1M requests and 400,000 GB-seconds per month

**Optimization Tips:**
- Use appropriate memory allocation
- Optimize code for faster execution
- Use provisioned concurrency for consistent performance
- Monitor and analyze cost patterns

**Step 18: Troubleshooting Common Issues**

**Function Not Executing:**
- Check IAM permissions
- Verify trigger configuration
- Review CloudWatch logs for errors

**Timeout Errors:**
- Increase timeout setting
- Optimize code performance
- Check for infinite loops

**Memory Errors:**
- Increase memory allocation
- Optimize memory usage in code
- Check for memory leaks

**Step 19: Integration with Other Services**

**API Gateway Integration:**
1. Create API Gateway REST API
2. Create resource and method
3. Set integration type to Lambda function
4. Deploy API to stage

**DynamoDB Integration:**
```python
import boto3

def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('MyTable')
    
    # Insert data
    table.put_item(Item={'id': '123', 'message': 'Hello World'})
    
    return {'statusCode': 200, 'body': 'Data saved'}
```

**Step 20: Cleanup**
When finished with testing:
1. **Delete function**:
   - Functions → Select function → Actions → Delete
   - Type "delete" to confirm
2. **Delete IAM role** (if no longer needed):
   - IAM → Roles → Select role → Delete
3. **Delete CloudWatch log group**:
   - CloudWatch → Log groups → Select → Delete

**Verification Checklist:**
- ✅ Lambda function created with Python runtime
- ✅ Function code deployed and saved
- ✅ Test event created and executed successfully
- ✅ Function logs visible in CloudWatch
- ✅ Function returns expected JSON response
- ✅ Error handling implemented
- ✅ Function performance monitored
- ✅ Understanding of Lambda pricing model
- ✅ Basic security practices applied
- ✅ Integration possibilities explored

### Task 13: SNS Topic
**Create an SNS topic and subscription to send email notifications.**

**Console-Based Solution:**

**Step 1: Navigate to SNS Console**
1. Go to **Amazon SNS Console**
2. Click "Topics" in left navigation
3. Click "Create topic"

**Step 2: Configure Topic Settings**
1. **Topic type**: Standard (for high throughput, at-least-once delivery)
   - Alternative: FIFO (for exactly-once delivery, ordered messages)
2. **Topic details**:
   - **Name**: "MyNotifications"
   - **Display name**: "My Application Notifications" (shown in SMS/email)
   - **Description**: "Topic for application alerts and notifications"

**Step 3: Access Policy (Advanced)**
1. **Access policy**: Choose configuration method
   - **Basic**: Use visual editor (recommended for beginners)
   - **JSON**: Write custom policy

**Basic Policy Configuration:**
1. **Define who can publish to this topic**:
   - **Publishers**: Only the topic owner
   - **AWS account IDs**: Leave blank (topic owner only)
2. **Define who can subscribe to this topic**:
   - **Subscribers**: Only the topic owner  
   - **AWS account IDs**: Leave blank

**Step 4: Delivery Policy and Retry Settings**
1. **Delivery retry policy**: Default (3 retries for HTTP/HTTPS)
2. **Dead letter queue**: None (for now)
3. **Content-based message deduplication**: Disabled (for Standard topics)

**Step 5: Encryption (Optional but Recommended)**
1. **Encryption**: Enable
2. **Customer managed CMK**: Select AWS managed key (free)
   - Or create custom KMS key for additional control

**Step 6: Tags (Optional)**
1. Add tags for organization:
   - Key: Environment, Value: Development
   - Key: Application, Value: NotificationSystem
   - Key: Owner, Value: DevTeam

**Step 7: Create the Topic**
1. Review all settings
2. Click "Create topic"
3. **Important**: Copy the Topic ARN (e.g., arn:aws:sns:us-east-1:123456789012:MyNotifications)

**Step 8: Create Email Subscription**
1. In the topic details page, click "Create subscription"
2. **Subscription configuration**:
   - **Protocol**: Email
   - **Endpoint**: your-email@example.com
   - **Enable raw message delivery**: Leave unchecked (for formatted messages)
3. Click "Create subscription"
4. **Important**: Check your email and click "Confirm subscription"
5. Subscription status should change from "Pending confirmation" to "Confirmed"

**Step 9: Create SMS Subscription (Optional)**
1. Click "Create subscription" again
2. **Subscription configuration**:
   - **Protocol**: SMS
   - **Endpoint**: +1234567890 (include country code)
3. Click "Create subscription"
4. **Note**: SMS subscriptions are automatically confirmed

**Step 10: Create Additional Subscription Types**

**Lambda Function Subscription:**
1. Create subscription:
   - **Protocol**: AWS Lambda
   - **Endpoint**: Select existing Lambda function
2. Lambda function will be triggered when messages are published

**SQS Queue Subscription:**
1. Create subscription:
   - **Protocol**: Amazon SQS
   - **Endpoint**: Select existing SQS queue
2. Messages will be delivered to the queue

**HTTP/HTTPS Webhook:**
1. Create subscription:
   - **Protocol**: HTTP or HTTPS
   - **Endpoint**: https://your-webhook-url.com/notifications
2. SNS will POST messages to your endpoint

**Step 11: Test the Topic by Publishing a Message**
1. In topic details, click "Publish message"
2. **Message details**:
   - **Subject**: "Test Notification from SNS"
   - **Message body**: Choose format
     - **Identical payload for all delivery protocols**
     - **Custom payload for each delivery protocol**

**Simple Message Example:**
```
Subject: Test Notification
Message body: 
Hello! This is a test notification from Amazon SNS. 
If you receive this message, your email subscription is working correctly.

Time: 2025-10-22 14:30:00 UTC
Source: AWS SNS Console Test
```

**Custom Payload Example:**
```json
{
  "default": "This is the default message",
  "email": "Hello!\n\nThis is a test email notification from SNS.\n\nBest regards,\nYour Application",
  "sms": "SNS Test: Your notification system is working!",
  "lambda": "{\"messageType\": \"test\", \"timestamp\": \"2025-10-22T14:30:00Z\"}"
}
```

3. Click "Publish message"

**Step 12: Verify Message Delivery**
1. **Check email inbox** for the notification
2. **Check phone** for SMS (if subscribed)
3. **Check Lambda logs** (if Lambda subscription exists)
4. **Monitor delivery status** in SNS console

**Step 13: Configure Message Attributes (Advanced)**
When publishing messages, you can add attributes:
1. Click "Message attributes" section
2. Add custom attributes:
   - **Name**: priority, **Type**: String, **Value**: high
   - **Name**: category, **Type**: String, **Value**: alert
   - **Name**: timestamp, **Type**: Number, **Value**: 1729604400

**Step 14: Set Up Message Filtering**
1. Edit any subscription
2. **Subscription filter policy**: Add JSON filter
```json
{
  "priority": ["high", "critical"],
  "category": ["alert", "warning"]
}
```
3. This subscription will only receive messages with matching attributes

**Step 15: Configure Delivery Status Logging**
1. In topic configuration, click "Edit"
2. **Delivery status logging**: Configure for different protocols
   - **Lambda delivery logging**: Create CloudWatch log group
   - **Email delivery logging**: Enable success and failure logging
   - **SMS delivery logging**: Track delivery attempts

**Step 16: Monitor Topic Metrics**
1. Click "Monitoring" tab in topic details
2. View CloudWatch metrics:
   - **NumberOfMessagesPublished**: Messages sent to topic
   - **NumberOfNotificationDelivered**: Successful deliveries
   - **NumberOfNotificationFailed**: Failed deliveries
   - **PublishSize**: Average message size

**Step 17: Create CloudWatch Alarms for SNS**
1. Go to **CloudWatch Console** → Alarms
2. Create alarm for failed notifications:
   - **Metric**: SNS → Topic Metrics → NumberOfNotificationsFailed
   - **Statistic**: Sum
   - **Period**: 5 minutes
   - **Threshold**: Greater than 0
   - **Action**: Send notification to another SNS topic

**Step 18: Integrate with Other AWS Services**

**CloudWatch Alarms Integration:**
1. In CloudWatch, create an alarm
2. **Actions**: Send notification to SNS topic
3. **SNS topic**: Select "MyNotifications"
4. Alarm notifications will be sent via email/SMS

**Lambda Function Publishing:**
```python
import boto3
import json

def lambda_handler(event, context):
    sns = boto3.client('sns')
    
    message = {
        'default': 'Application event occurred',
        'email': 'An important event happened in your application.',
        'sms': 'App Alert: Check your dashboard'
    }
    
    response = sns.publish(
        TopicArn='arn:aws:sns:us-east-1:123456789012:MyNotifications',
        Message=json.dumps(message),
        Subject='Application Alert',
        MessageStructure='json',
        MessageAttributes={
            'priority': {
                'DataType': 'String',
                'StringValue': 'high'
            }
        }
    )
    
    return response
```

**Step 19: Security Best Practices**

**Topic Access Control:**
1. Use IAM policies to control who can publish/subscribe
2. Example policy for publish-only access:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sns:Publish",
            "Resource": "arn:aws:sns:us-east-1:123456789012:MyNotifications"
        }
    ]
}
```

**Subscription Confirmation:**
- Always confirm email subscriptions manually
- Use confirmed endpoints only
- Regularly review and clean up unused subscriptions

**Message Content Security:**
- Don't include sensitive data in messages
- Use encryption for sensitive notifications
- Implement message signing for verification

**Step 20: Cost Optimization**

**SNS Pricing:**
- **Requests**: $0.50 per 1 million requests (first 1M free)
- **Email notifications**: $2.00 per 100,000 notifications
- **SMS notifications**: Varies by country (typically $0.75 per 100 in US)
- **Mobile push**: $0.50 per 1 million notifications

**Cost Optimization Tips:**
- Use message filtering to reduce unnecessary notifications
- Combine multiple updates into digest notifications
- Use SQS for high-volume, less time-sensitive notifications

**Step 21: Advanced Features**

**Dead Letter Queues:**
1. Create SQS queue for failed deliveries
2. In subscription settings, configure DLQ
3. Analyze failed messages for troubleshooting

**Message Deduplication (FIFO topics):**
1. Create FIFO topic (name must end with .fifo)
2. Enable content-based deduplication
3. Ensures exactly-once delivery

**Cross-Region Replication:**
1. Create similar topics in multiple regions
2. Use Lambda or application logic to publish to multiple regions
3. Provides disaster recovery for notifications

**Step 22: Troubleshooting Common Issues**

**Email Not Received:**
- Check spam/junk folder
- Verify email address is correct
- Confirm subscription is confirmed
- Check topic delivery status logs

**SMS Not Delivered:**
- Verify phone number format (+country code)
- Check SMS spending limits in account settings
- Verify destination country supports SMS delivery

**High Costs:**
- Review message volume and frequency
- Implement message filtering
- Consider using SQS for non-urgent notifications

**Step 23: Automation with Infrastructure as Code**

**CloudFormation Template:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyNotificationsTopic:
    Type: AWS::SNS::Topic
    Properties:
      TopicName: MyNotifications
      DisplayName: My Application Notifications
      
  EmailSubscription:
    Type: AWS::SNS::Subscription
    Properties:
      Protocol: email
      TopicArn: !Ref MyNotificationsTopic
      Endpoint: your-email@example.com
```

**Step 24: Cleanup**
When finished with testing:
1. **Delete subscriptions** first:
   - Select subscription → Delete
2. **Delete topic**:
   - Topics → Select topic → Delete
   - Type topic name to confirm
3. **Review CloudWatch logs** and delete if not needed
4. **Check billing** for any SMS charges

**Verification Checklist:**
- ✅ SNS topic created with appropriate settings
- ✅ Email subscription confirmed and working
- ✅ Test message published and received
- ✅ Message filtering configured (if needed)
- ✅ CloudWatch monitoring enabled
- ✅ Security policies properly configured
- ✅ Cost implications understood
- ✅ Integration with other services tested
- ✅ Troubleshooting procedures documented

### Task 14: CloudFormation Template
**Write a basic CloudFormation template to create a VPC with public and private subnets.**

**Console-Based Solution:**

**Step 1: Navigate to CloudFormation Console**
1. Go to **AWS CloudFormation Console**
2. Click "Stacks" in left navigation
3. Click "Create stack" → "With new resources (standard)"

**Step 2: Prepare CloudFormation Template**
Create a file named `vpc-template.yaml` on your local machine with the following content:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Complete VPC with public and private subnets, NAT Gateway, and proper routing'

Parameters:
  VpcCidr:
    Type: String
    Default: '10.0.0.0/16'
    Description: 'CIDR block for the VPC'
    AllowedPattern: '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/(1[6-9]|2[0-8]))$'
    
  Environment:
    Type: String
    Default: 'development'
    AllowedValues: 
      - development
      - staging
      - production
    Description: 'Environment name for resource tagging'

  ProjectName:
    Type: String
    Default: 'MyProject'
    Description: 'Project name for resource naming'

Resources:
  # VPC
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-VPC'
        - Key: Environment
          Value: !Ref Environment
        - Key: Project
          Value: !Ref ProjectName

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-IGW'
        - Key: Environment
          Value: !Ref Environment

  # Attach Internet Gateway to VPC
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref MyVPC
      InternetGatewayId: !Ref InternetGateway

  # Public Subnet in AZ-a
  PublicSubnetA:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: '10.0.1.0/24'
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-Public-Subnet-A'
        - Key: Type
          Value: 'Public'
        - Key: Environment
          Value: !Ref Environment

  # Public Subnet in AZ-b (for high availability)
  PublicSubnetB:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: '10.0.2.0/24'
      AvailabilityZone: !Select [1, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-Public-Subnet-B'
        - Key: Type
          Value: 'Public'
        - Key: Environment
          Value: !Ref Environment

  # Private Subnet in AZ-a
  PrivateSubnetA:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: '10.0.10.0/24'
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-Private-Subnet-A'
        - Key: Type
          Value: 'Private'
        - Key: Environment
          Value: !Ref Environment

  # Private Subnet in AZ-b
  PrivateSubnetB:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      CidrBlock: '10.0.11.0/24'
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-Private-Subnet-B'
        - Key: Type
          Value: 'Private'
        - Key: Environment
          Value: !Ref Environment

  # Elastic IP for NAT Gateway
  NATGatewayEIP:
    Type: AWS::EC2::EIP
    DependsOn: AttachGateway
    Properties:
      Domain: vpc
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-NAT-EIP'

  # NAT Gateway (in public subnet for internet access)
  NATGateway:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NATGatewayEIP.AllocationId
      SubnetId: !Ref PublicSubnetA
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-NAT-Gateway'

  # Public Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-Public-RT'
        - Key: Type
          Value: 'Public'

  # Private Route Table
  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref MyVPC
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-Private-RT'
        - Key: Type
          Value: 'Private'

  # Public Route (to Internet Gateway)
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: '0.0.0.0/0'
      GatewayId: !Ref InternetGateway

  # Private Route (to NAT Gateway)
  PrivateRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable
      DestinationCidrBlock: '0.0.0.0/0'
      NatGatewayId: !Ref NATGateway

  # Associate Public Subnets with Public Route Table
  PublicSubnetARouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnetA
      RouteTableId: !Ref PublicRouteTable

  PublicSubnetBRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnetB
      RouteTableId: !Ref PublicRouteTable

  # Associate Private Subnets with Private Route Table
  PrivateSubnetARouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateSubnetA
      RouteTableId: !Ref PrivateRouteTable

  PrivateSubnetBRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateSubnetB
      RouteTableId: !Ref PrivateRouteTable

  # Security Group for Web Servers (in public subnet)
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Security group for web servers'
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: '0.0.0.0/0'
          Description: 'HTTP access from anywhere'
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: '0.0.0.0/0'
          Description: 'HTTPS access from anywhere'
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: '10.0.0.0/16'
          Description: 'SSH access from VPC'
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-Web-SG'

  # Security Group for Database Servers (in private subnet)
  DatabaseSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Security group for database servers'
      VpcId: !Ref MyVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref WebServerSecurityGroup
          Description: 'MySQL access from web servers'
        - IpProtocol: tcp
          FromPort: 5432
          ToPort: 5432
          SourceSecurityGroupId: !Ref WebServerSecurityGroup
          Description: 'PostgreSQL access from web servers'
      Tags:
        - Key: Name
          Value: !Sub '${ProjectName}-${Environment}-DB-SG'

Outputs:
  VPCId:
    Description: 'VPC ID'
    Value: !Ref MyVPC
    Export:
      Name: !Sub '${AWS::StackName}-VPC-ID'

  VPCCidr:
    Description: 'VPC CIDR Block'
    Value: !Ref VpcCidr
    Export:
      Name: !Sub '${AWS::StackName}-VPC-CIDR'

  PublicSubnetAId:
    Description: 'Public Subnet A ID'
    Value: !Ref PublicSubnetA
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnetA-ID'

  PublicSubnetBId:
    Description: 'Public Subnet B ID'
    Value: !Ref PublicSubnetB
    Export:
      Name: !Sub '${AWS::StackName}-PublicSubnetB-ID'

  PrivateSubnetAId:
    Description: 'Private Subnet A ID'
    Value: !Ref PrivateSubnetA
    Export:
      Name: !Sub '${AWS::StackName}-PrivateSubnetA-ID'

  PrivateSubnetBId:
    Description: 'Private Subnet B ID'
    Value: !Ref PrivateSubnetB
    Export:
      Name: !Sub '${AWS::StackName}-PrivateSubnetB-ID'

  WebServerSecurityGroupId:
    Description: 'Web Server Security Group ID'
    Value: !Ref WebServerSecurityGroup
    Export:
      Name: !Sub '${AWS::StackName}-WebSG-ID'

  DatabaseSecurityGroupId:
    Description: 'Database Security Group ID'
    Value: !Ref DatabaseSecurityGroup
    Export:
      Name: !Sub '${AWS::StackName}-DatabaseSG-ID'

  NATGatewayIP:
    Description: 'NAT Gateway Elastic IP'
    Value: !Ref NATGatewayEIP
    Export:
      Name: !Sub '${AWS::StackName}-NAT-IP'
```

**Step 3: Upload Template to CloudFormation**
1. **Template source**: Upload a template file
2. Click "Choose file" and select your `vpc-template.yaml`
3. Click "Next"

**Step 4: Configure Stack Details**
1. **Stack name**: "my-vpc-infrastructure"
2. **Parameters**:
   - **VpcCidr**: 10.0.0.0/16 (default)
   - **Environment**: development
   - **ProjectName**: MyWebApp
3. Click "Next"

**Step 5: Configure Stack Options**
1. **Tags** (optional but recommended):
   - Key: Owner, Value: DevTeam
   - Key: CostCenter, Value: Engineering
   - Key: Purpose, Value: NetworkInfrastructure
2. **Permissions**: Use default service role (or create if needed)
3. **Stack failure options**: Roll back all stack resources
4. **Advanced options**: Leave defaults
5. Click "Next"

**Step 6: Review and Create**
1. Review all configuration details
2. **Capabilities**: Check "I acknowledge that AWS CloudFormation might create IAM resources"
3. Click "Submit"

**Step 7: Monitor Stack Creation**
1. **Stack status** will show progression:
   - CREATE_IN_PROGRESS
   - CREATE_COMPLETE (success)
   - CREATE_FAILED (if errors occur)
2. **Events** tab shows detailed progress
3. **Resources** tab lists all created resources
4. Creation typically takes 3-5 minutes

**Step 8: Verify Stack Outputs**
1. Click "Outputs" tab
2. Note the exported values:
   - VPC ID
   - Subnet IDs
   - Security Group IDs
   - NAT Gateway IP
3. These can be referenced by other stacks

**Step 9: Test the VPC Infrastructure**

**Launch Test Instance in Public Subnet:**
1. Go to EC2 Console → Launch instance
2. Select Amazon Linux 2023 AMI
3. **Network settings**:
   - VPC: Select your CloudFormation-created VPC
   - Subnet: Select public subnet A
   - Auto-assign public IP: Enable
   - Security group: Select web server security group
4. Launch and test internet connectivity

**Launch Test Instance in Private Subnet:**
1. Launch another instance
2. **Network settings**:
   - VPC: Your CloudFormation VPC
   - Subnet: Select private subnet A
   - Auto-assign public IP: Disable
   - Security group: Database security group
3. Test that it can reach internet via NAT Gateway

**Step 10: Update Stack (Demonstrate Stack Updates)**
1. Modify the template (e.g., add a new subnet)
2. In CloudFormation → Stacks → Select your stack
3. Click "Update"
4. **Replace current template**: Upload modified template
5. Review changes and update

**Step 11: Use Stack Outputs in Another Template**
Create a simple EC2 template that references your VPC:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'EC2 instance using existing VPC'

Parameters:
  VPCStackName:
    Type: String
    Default: 'my-vpc-infrastructure'
    Description: 'Name of the VPC stack to import values from'

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0abcdef1234567890  # Update with current AMI ID
      InstanceType: t3.micro
      SubnetId: 
        Fn::ImportValue: !Sub '${VPCStackName}-PublicSubnetA-ID'
      SecurityGroupIds:
        - Fn::ImportValue: !Sub '${VPCStackName}-WebSG-ID'
      Tags:
        - Key: Name
          Value: 'CloudFormation Web Server'
```

**Step 12: Stack Management Best Practices**

**Stack Organization:**
- Use descriptive stack names
- Group related resources in logical stacks
- Use nested stacks for complex architectures
- Export values for cross-stack references

**Version Control:**
- Store templates in Git repository
- Use semantic versioning for templates
- Document changes in template descriptions
- Review templates before deployment

**Parameter Management:**
- Use parameter files for different environments
- Validate parameter constraints
- Provide meaningful descriptions
- Use appropriate parameter types

**Step 13: Monitor Stack Costs**
1. Go to **Cost Explorer**
2. Filter by Tag: AWS::CloudFormation::StackName
3. View costs for your specific stack
4. Set up budget alerts for stack spending

**Step 14: Troubleshooting Stack Issues**

**Common Creation Failures:**
- **Insufficient IAM permissions**: Add required CloudFormation permissions
- **Resource limits exceeded**: Check service quotas
- **Invalid parameter values**: Verify CIDR blocks don't overlap
- **Availability zone issues**: Ensure AZs support required resources

**Debug Steps:**
1. Check Events tab for specific error messages
2. Review resource dependencies
3. Validate template syntax
4. Test with minimal template first

**Step 15: Advanced CloudFormation Features**

**Conditions:**
```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, 'production']

Resources:
  ExpensiveResource:
    Type: AWS::EC2::Instance
    Condition: IsProduction
```

**Mappings:**
```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0abcdef1234567890
    us-west-2:
      AMI: ami-1234567890abcdef0
```

**Functions:**
```yaml
!Join ['-', [!Ref ProjectName, !Ref Environment, 'resource']]
!Select [0, !GetAZs '']
!Sub '${ProjectName}-${Environment}-${AWS::Region}'
```

**Step 16: Cleanup Stack**
When finished with testing:
1. **Delete dependent resources** first (EC2 instances, etc.)
2. In CloudFormation → Stacks → Select stack
3. Click "Delete"
4. Confirm deletion
5. Monitor deletion progress in Events tab
6. **Note**: NAT Gateway and Elastic IP will incur charges until deleted

**Verification Checklist:**
- ✅ CloudFormation template created with proper syntax
- ✅ VPC with public and private subnets deployed
- ✅ Internet Gateway and NAT Gateway configured
- ✅ Route tables properly associated
- ✅ Security groups created with appropriate rules
- ✅ Stack outputs exported for reuse
- ✅ Infrastructure tested with EC2 instances
- ✅ Stack update process demonstrated
- ✅ Cost monitoring configured
- ✅ Cleanup procedures documented

### Task 15: Cost Optimization
**Identify and implement cost optimization strategies for your AWS resources.**

**Console-Based Solution:**

**Step 1: Access AWS Cost Management Tools**
1. Navigate to **AWS Billing and Cost Management Console**
2. In left navigation, explore cost management tools:
   - **Cost Explorer**: Analyze spending patterns
   - **Budgets**: Set spending alerts
   - **Cost Anomaly Detection**: Identify unusual spending
   - **AWS Cost and Usage Reports**: Detailed billing data
   - **Savings Plans**: Commitment-based discounts

**Step 2: Enable Cost Explorer**
1. Go to **Cost Explorer**
2. If first time: Click "Enable Cost Explorer"
3. **Note**: Takes 24 hours to populate initial data
4. Cost Explorer provides 12 months of historical data

**Step 3: Create Cost Budget**
1. Go to **Budgets** → "Create budget"
2. **Budget setup**:
   - **Budget type**: Cost budget
   - **Budget name**: "Monthly-AWS-Budget"
   - **Period**: Monthly
   - **Budget renewal type**: Recurring budget
   - **Start month**: Current month
3. **Budget amount**: $100 (adjust based on your expected usage)
4. **Budget scope**:
   - **Filters**: All AWS services (or specific services)
   - **Accounts**: Current account
   - **Regions**: All regions (or specific regions)

**Step 4: Configure Budget Alerts**
1. **Alert #1 - 80% threshold**:
   - **Alert threshold**: 80% of budgeted amount
   - **Trigger**: Actual costs
   - **Email recipients**: your-email@example.com
2. **Alert #2 - 100% threshold**:
   - **Alert threshold**: 100% of budgeted amount  
   - **Trigger**: Forecasted costs
   - **Email recipients**: finance-team@example.com
3. **Alert #3 - 120% threshold**:
   - **Alert threshold**: 120% of budgeted amount
   - **Trigger**: Actual costs
   - **Email recipients**: management@example.com

**Step 5: Identify Unused EBS Volumes**
1. Go to **EC2 Console** → "Volumes"
2. **Filter by State**: Available (these are unattached volumes)
3. Review list for volumes not attached to instances:
   - **Volume ID**: Note the IDs
   - **Size**: Check size for cost impact
   - **Created**: Look for older volumes (likely unused)
   - **Snapshot**: Verify if snapshots exist before deleting
4. **Action Plan**:
   - Create snapshots of important unattached volumes
   - Delete unused volumes after confirmation

**Step 6: Find Unattached Elastic IPs**
1. Go to **EC2 Console** → "Elastic IPs"
2. Look for IPs with **Instance ID**: (empty)
3. **Associated instance**: Shows "None" for unattached IPs
4. **Cost impact**: Unattached EIPs are charged hourly
5. **Action Plan**:
   - Release unused Elastic IPs immediately
   - Associate needed IPs with instances

**Step 7: Analyze Instance Utilization**
1. Go to **CloudWatch Console** → "Metrics"
2. **EC2 Metrics** → "Per-Instance Metrics"
3. For each instance, check **CPUUtilization**:
   - **Time range**: Last 2 weeks
   - **Period**: 1 day
   - **Statistic**: Average
4. **Identify optimization opportunities**:
   - CPU < 10%: Consider downsizing instance type
   - CPU < 5%: Consider stopping during off-hours
   - Consistent low usage: Move to smaller instance type

**Step 8: Set Up Cost Anomaly Detection**
1. Go to **Cost Anomaly Detection**
2. Click "Create cost monitor"
3. **Monitor configuration**:
   - **Monitor name**: "Overall-Account-Monitor"
   - **Monitor type**: AWS services
   - **Services**: All AWS services
4. **Alert configuration**:
   - **Alert threshold**: $50 (adjust based on normal spending)
   - **Alert frequency**: Daily
   - **Email recipients**: Add your email

**Step 9: Implement S3 Storage Optimization**

**Enable S3 Intelligent-Tiering:**
1. Go to **S3 Console** → Select bucket
2. **Management** tab → "Create lifecycle rule"
3. **Lifecycle rule configuration**:
   - **Rule name**: "intelligent-tiering-rule"
   - **Choose rule scope**: Apply to all objects
   - **Lifecycle rule actions**: ✅ Transition current versions
4. **Transition settings**:
   - **Storage class transitions**: 
     - Day 0: Intelligent-Tiering
5. Click "Create rule"

**Create Comprehensive Lifecycle Policy:**
1. Create another lifecycle rule:
   - **Rule name**: "comprehensive-lifecycle"
   - **Object filters**: Prefix "documents/"
2. **Transitions**:
   - Day 30: Standard-Infrequent Access (IA)
   - Day 90: Glacier Flexible Retrieval
   - Day 365: Glacier Deep Archive
3. **Expiration**: Delete objects after 2555 days (7 years)

**Step 10: Optimize EC2 Instance Costs**

**Right-Size Instances:**
1. Go to **AWS Compute Optimizer** (if enabled)
2. Review instance recommendations:
   - **Under-provisioned**: Instances needing more resources
   - **Over-provisioned**: Instances with excess capacity
   - **Optimized**: Properly sized instances
3. **Implementation**:
   - Stop instance
   - Change instance type
   - Start instance

**Implement Instance Scheduling:**
1. **For Development/Test environments**:
   - Use AWS Instance Scheduler or Lambda functions
   - Stop instances during non-business hours
   - Example: Run 8 AM - 6 PM, Monday-Friday
2. **Potential savings**: 70% reduction for dev/test workloads

**Step 11: Reserved Instance Analysis**
1. Go to **EC2 Console** → "Reserved Instances"
2. Click "Purchase Reserved Instances"
3. **RI Recommendations**:
   - Review suggested RIs based on usage patterns
   - **1-year term**: ~20% savings vs On-Demand
   - **3-year term**: ~40% savings vs On-Demand
4. **Purchase strategy**:
   - Start with partial commitment (50-70% of steady workload)
   - Monitor usage before committing to additional RIs

**Step 12: Set Up Automated Cost Controls**

**Service Control Policies (for Organizations):**
1. Create policies to prevent expensive actions:
   - Restrict instance types (no large instances without approval)
   - Block certain regions
   - Require specific tags

**Billing Alerts via CloudWatch:**
1. Go to **CloudWatch** → "Alarms"
2. Create alarm:
   - **Metric**: Billing → Total Estimated Charge
   - **Threshold**: $80 (80% of budget)
   - **Action**: Send SNS notification

**Step 13: Implement Tagging Strategy for Cost Allocation**
1. **Required tags for all resources**:
   - **Environment**: production, staging, development
   - **Project**: project-name
   - **Owner**: team-name
   - **CostCenter**: department-code
2. **Enforce tagging**:
   - Use AWS Config rules
   - Implement tag policies (AWS Organizations)
   - Set up automated tagging via Lambda

**Step 14: Monitor and Optimize Data Transfer Costs**
1. **CloudFront for static content**:
   - Reduces data transfer costs
   - Improves performance globally
2. **VPC Endpoints for AWS services**:
   - Eliminates NAT Gateway data transfer charges
   - Direct private connectivity to S3, DynamoDB
3. **Regional optimization**:
   - Keep resources in same region when possible
   - Use regional endpoints for API calls

**Step 15: Database Cost Optimization**

**RDS Optimization:**
1. **Right-size instances** based on CloudWatch metrics
2. **Use read replicas** instead of larger primary instances
3. **Reserved Instances** for predictable workloads
4. **Storage optimization**:
   - Use gp3 instead of gp2 for better price/performance
   - Enable storage autoscaling to avoid over-provisioning

**DynamoDB Optimization:**
1. **Use On-Demand** for unpredictable workloads
2. **Use Provisioned** with Auto Scaling for steady workloads
3. **Enable DynamoDB Global Tables** only when needed
4. **Use DynamoDB DAX** only for microsecond latency requirements

**Step 16: Create Cost Optimization Dashboard**
1. **CloudWatch Dashboard**:
   - Daily spending trend
   - Top 5 services by cost
   - Instance utilization metrics
   - Storage growth trends
2. **Custom metrics**:
   - Cost per customer
   - Cost per transaction
   - Monthly cost trends

**Step 17: Implement FinOps Best Practices**

**Regular Cost Reviews:**
- Weekly: Review budget alerts and anomalies
- Monthly: Analyze cost trends and optimization opportunities
- Quarterly: Review Reserved Instance utilization and renewals

**Cost Allocation:**
- Use detailed tagging for chargeback/showback
- Implement cost allocation tags
- Create department-specific budgets

**Automation:**
- Automate resource cleanup (unused resources)
- Implement infrastructure as code with cost controls
- Use AWS Config for compliance and cost governance

**Step 18: Advanced Cost Optimization Tools**

**AWS Trusted Advisor (Business/Enterprise Support):**
1. **Cost Optimization checks**:
   - Unattached EBS volumes
   - Underutilized instances
   - Idle load balancers
   - Unassociated Elastic IPs

**AWS Well-Architected Tool:**
1. Use Cost Optimization pillar
2. Regular workload reviews
3. Implement recommended improvements

**Third-party tools:**
- CloudHealth
- CloudCheckr
- ParkMyCloud (for instance scheduling)

**Step 19: Emergency Cost Controls**

**Immediate cost reduction steps:**
1. **Stop all non-production instances**
2. **Release unattached Elastic IPs**
3. **Delete unused load balancers**
4. **Suspend Auto Scaling groups** (if safe)
5. **Reduce RDS instance sizes** temporarily

**Billing alerts escalation:**
1. Create multiple threshold alerts (50%, 80%, 100%, 120%)
2. Different notification lists for each threshold
3. Include phone/SMS notifications for critical thresholds

**Step 20: Cost Optimization Checklist**

**Monthly Review:**
- ✅ Review budget vs actual spending
- ✅ Analyze top 10 cost contributors
- ✅ Check for unused resources
- ✅ Review Reserved Instance utilization
- ✅ Optimize storage lifecycle policies
- ✅ Update cost allocation tags

**Quarterly Review:**
- ✅ Reserved Instance renewal decisions
- ✅ Architecture review for cost optimization
- ✅ Update budgets based on business changes
- ✅ Review and update cost controls

**Verification Checklist:**
- ✅ Cost Explorer enabled and configured
- ✅ Monthly budget created with appropriate alerts
- ✅ Unused EBS volumes identified and cleaned up
- ✅ Unattached Elastic IPs released
- ✅ Instance utilization analyzed and optimized
- ✅ S3 lifecycle policies implemented
- ✅ Cost anomaly detection configured
- ✅ Tagging strategy implemented for cost allocation
- ✅ Reserved Instance strategy developed
- ✅ Automated cost controls established

## Intermediate Level Solutions (Tasks 16-35)

### Task 16: VPC Creation
**Create a custom VPC with public and private subnets across two availability zones.**

**Solution:**

**Console-Based Solution:**

**Step 1: Navigate to VPC Console**
1. Go to **VPC Console**
2. Click "Your VPCs" in left navigation
3. Click "Create VPC"

**Step 2: Configure VPC Settings**
1. **VPC settings**:
   - **Resources to create**: VPC and more (recommended for complete setup)
   - **Name tag**: "CustomVPC"
   - **IPv4 CIDR block**: 10.0.0.0/16
   - **IPv6 CIDR block**: No IPv6 CIDR block
   - **Tenancy**: Default

**Step 3: Configure Subnets**
1. **Number of Availability Zones**: 2
2. **Number of public subnets**: 2
3. **Number of private subnets**: 2
4. **Subnet CIDR blocks** (auto-calculated):
   - Public subnet 1: 10.0.1.0/24
   - Public subnet 2: 10.0.2.0/24
   - Private subnet 1: 10.0.10.0/24
   - Private subnet 2: 10.0.11.0/24

**Step 4: Configure NAT Gateways**
1. **NAT gateways**: 1 per AZ (recommended for high availability)
2. **VPC endpoints**: None (can add later if needed)

**Step 5: Configure DNS Settings**
1. **DNS options**:
   - ✅ Enable DNS hostnames
   - ✅ Enable DNS resolution

**Step 6: Additional Tags**
1. Add tags for organization:
   - Key: Environment, Value: Development
   - Key: Project, Value: NetworkInfrastructure
   - Key: Owner, Value: NetworkTeam

**Step 7: Create VPC**
1. Review all settings in preview
2. Click "Create VPC"
3. Wait for creation to complete (2-3 minutes)

**Step 8: Verify VPC Creation**
1. **VPC Dashboard** should show:
   - 1 VPC created
   - 4 subnets (2 public, 2 private)
   - 1 Internet Gateway
   - 2 NAT Gateways
   - Route tables configured automatically

**Verification Checklist:**
- ✅ VPC created with correct CIDR block
- ✅ Public and private subnets in multiple AZs
- ✅ Internet Gateway attached and configured
- ✅ NAT Gateways deployed for private subnet internet access
- ✅ Route tables properly configured
- ✅ DNS resolution and hostnames enabled

### Task 17: NAT Gateway Setup
**Configure NAT Gateway for private subnet internet access.**

**Solution:**
```bash
# Allocate Elastic IPs for NAT Gateways
EIP_1=$(aws ec2 allocate-address \
    --domain vpc \
    --query 'AllocationId' \
    --output text)

EIP_2=$(aws ec2 allocate-address \
    --domain vpc \
    --query 'AllocationId' \
    --output text)

# Create NAT Gateways in public subnets
NAT_GW_1=$(aws ec2 create-nat-gateway \
    --subnet-id $PUBLIC_SUBNET_1 \
    --allocation-id $EIP_1 \
    --query 'NatGateway.NatGatewayId' \
    --output text)

NAT_GW_2=$(aws ec2 create-nat-gateway \
    --subnet-id $PUBLIC_SUBNET_2 \
    --allocation-id $EIP_2 \
    --query 'NatGateway.NatGatewayId' \
    --output text)

# Wait for NAT Gateways to be available
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_1
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_2

# Create route tables for private subnets
PRIVATE_RT_1=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' \
    --output text)

PRIVATE_RT_2=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' \
    --output text)

# Add routes to NAT Gateways
aws ec2 create-route \
    --route-table-id $PRIVATE_RT_1 \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id $NAT_GW_1

aws ec2 create-route \
    --route-table-id $PRIVATE_RT_2 \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id $NAT_GW_2

# Associate private subnets with their route tables
aws ec2 associate-route-table \
    --subnet-id $PRIVATE_SUBNET_1 \
    --route-table-id $PRIVATE_RT_1

aws ec2 associate-route-table \
    --subnet-id $PRIVATE_SUBNET_2 \
    --route-table-id $PRIVATE_RT_2

# Tag NAT Gateways
aws ec2 create-tags \
    --resources $NAT_GW_1 \
    --tags Key=Name,Value=NAT-Gateway-1

aws ec2 create-tags \
    --resources $NAT_GW_2 \
    --tags Key=Name,Value=NAT-Gateway-2

echo "NAT Gateway 1: $NAT_GW_1"
echo "NAT Gateway 2: $NAT_GW_2"
```

### Task 18: Application Load Balancer
**Create an ALB to distribute traffic between multiple EC2 instances.**

**Solution:**
```bash
# Create security group for ALB
ALB_SG=$(aws ec2 create-security-group \
    --group-name alb-security-group \
    --description "Security group for Application Load Balancer" \
    --vpc-id $VPC_ID \
    --query 'GroupId' \
    --output text)

# Allow HTTP and HTTPS traffic
aws ec2 authorize-security-group-ingress \
    --group-id $ALB_SG \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $ALB_SG \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# Create Application Load Balancer
ALB_ARN=$(aws elbv2 create-load-balancer \
    --name my-application-load-balancer \
    --subnets $PUBLIC_SUBNET_1 $PUBLIC_SUBNET_2 \
    --security-groups $ALB_SG \
    --query 'LoadBalancers[0].LoadBalancerArn' \
    --output text)

# Create target group
TG_ARN=$(aws elbv2 create-target-group \
    --name my-target-group \
    --protocol HTTP \
    --port 80 \
    --vpc-id $VPC_ID \
    --health-check-path / \
    --health-check-interval-seconds 30 \
    --health-check-timeout-seconds 5 \
    --healthy-threshold-count 2 \
    --unhealthy-threshold-count 5 \
    --query 'TargetGroups[0].TargetGroupArn' \
    --output text)

# Create listener
aws elbv2 create-listener \
    --load-balancer-arn $ALB_ARN \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=$TG_ARN

# Launch EC2 instances for the target group
USER_DATA=$(base64 << 'EOF'
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Web Server $(hostname -f)</h1>" > /var/www/html/index.html
EOF
)

# Create security group for web servers
WEB_SG=$(aws ec2 create-security-group \
    --group-name web-servers-sg \
    --description "Security group for web servers" \
    --vpc-id $VPC_ID \
    --query 'GroupId' \
    --output text)

# Allow HTTP traffic from ALB
aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG \
    --protocol tcp \
    --port 80 \
    --source-group $ALB_SG

# Launch instances
INSTANCE_1=$(aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --count 1 \
    --instance-type t3.micro \
    --key-name my-key-pair \
    --security-group-ids $WEB_SG \
    --subnet-id $PRIVATE_SUBNET_1 \
    --user-data "$USER_DATA" \
    --query 'Instances[0].InstanceId' \
    --output text)

INSTANCE_2=$(aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --count 1 \
    --instance-type t3.micro \
    --key-name my-key-pair \
    --security-group-ids $WEB_SG \
    --subnet-id $PRIVATE_SUBNET_2 \
    --user-data "$USER_DATA" \
    --query 'Instances[0].InstanceId' \
    --output text)

# Wait for instances to be running
aws ec2 wait instance-running --instance-ids $INSTANCE_1 $INSTANCE_2

# Register instances with target group
aws elbv2 register-targets \
    --target-group-arn $TG_ARN \
    --targets Id=$INSTANCE_1 Id=$INSTANCE_2

# Get ALB DNS name
ALB_DNS=$(aws elbv2 describe-load-balancers \
    --load-balancer-arns $ALB_ARN \
    --query 'LoadBalancers[0].DNSName' \
    --output text)

echo "Application Load Balancer DNS: $ALB_DNS"
echo "Target Group ARN: $TG_ARN"
echo "Instances: $INSTANCE_1, $INSTANCE_2"
```

### Task 19: Auto Scaling Group
**Set up an Auto Scaling Group with launch templates and scaling policies.**

**Console-Based Solution:**

**Step 1: Create Launch Template**
1. Go to **EC2 Console** → "Launch Templates"
2. Click "Create launch template"
3. **Template configuration**:
   - **Name**: "web-server-template"
   - **Description**: "Template for web server auto scaling"
4. **Application and OS Images**: Amazon Linux 2023
5. **Instance type**: t3.micro
6. **Key pair**: Select existing key pair
7. **Security groups**: Select web server security group
8. **User data** (optional):
   ```bash
   #!/bin/bash
   yum update -y
   yum install -y httpd
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Web Server $(hostname -f)</h1>" > /var/www/html/index.html
   ```
9. Click "Create launch template"

**Step 2: Create Auto Scaling Group**
1. Go to **Auto Scaling Groups** → "Create Auto Scaling group"
2. **Group details**:
   - **Name**: "web-server-asg"
   - **Launch template**: Select "web-server-template"
3. **Network**:
   - **VPC**: Select your VPC
   - **Subnets**: Select public subnets across multiple AZs
4. **Instance configuration**:
   - **Desired capacity**: 2
   - **Minimum capacity**: 1
   - **Maximum capacity**: 4

**Step 3: Configure Scaling Policies**
1. **Target tracking scaling policies**:
   - **Policy name**: "cpu-target-tracking"
   - **Metric type**: Average CPU utilization
   - **Target value**: 70%
2. **Instance refresh**: Enable for rolling updates

**Verification Checklist:**
- ✅ Launch template created with proper configuration
- ✅ Auto Scaling Group spanning multiple AZs
- ✅ Scaling policies configured for CPU utilization
- ✅ Health checks configured properly

### Task 20: RDS Multi-AZ
**Configure RDS with Multi-AZ deployment and automated backups.**

**Console-Based Solution:**

**Step 1: Create Multi-AZ RDS Instance**
1. Go to **RDS Console** → "Create database"
2. **Engine**: MySQL 8.0
3. **Templates**: Production
4. **Availability and durability**:
   - ✅ **Multi-AZ DB instance**
5. **Settings**:
   - **DB instance identifier**: "prod-mysql-multiaz"
   - **Master username**: "admin"
   - **Password**: Auto-generate or set manually
6. **Instance configuration**: db.t3.small or larger
7. **Storage**: 100 GB with autoscaling enabled
8. **Backup**:
   - **Retention period**: 14 days
   - **Backup window**: 03:00-04:00 UTC
   - **Delete automated backups**: No

**Step 2: Configure Advanced Settings**
1. **Monitoring**: Enable Performance Insights
2. **Maintenance window**: Select off-peak hours
3. **Deletion protection**: Enable

**Verification Checklist:**
- ✅ Multi-AZ deployment configured
- ✅ Automated backups enabled (14 days retention)
- ✅ Performance monitoring enabled
- ✅ Deletion protection enabled

### Task 21: CloudFront Distribution
**Set up CloudFront to serve content from an S3 bucket with custom domain.**

**Console-Based Solution:**

**Step 1: Prepare S3 Bucket**
1. Create S3 bucket for CloudFront origin
2. Upload content files
3. Configure bucket policy for CloudFront access

**Step 2: Create CloudFront Distribution**
1. Go to **CloudFront Console** → "Create distribution"
2. **Origin settings**:
   - **Origin domain**: Select S3 bucket
   - **Origin access**: Origin access control settings (recommended)
3. **Cache behavior**:
   - **Allowed HTTP methods**: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
   - **Cache policy**: Managed-CachingOptimized
4. **Settings**:
   - **Price class**: Use all edge locations
   - **Alternate domain names**: your-domain.com
   - **SSL certificate**: Request or import certificate

**Verification Checklist:**
- ✅ CloudFront distribution created
- ✅ Origin access control configured
- ✅ Custom domain configured with SSL
- ✅ Cache behaviors optimized

### Task 22: IAM Roles and Policies
**Create custom IAM roles and policies following the principle of least privilege.**

**Console-Based Solution:**

**Step 1: Create Custom Policy**
1. Go to **IAM Console** → "Policies" → "Create policy"
2. **Service**: EC2
3. **Actions**: Specific actions (DescribeInstances, StartInstances, StopInstances)
4. **Resources**: Specific ARNs or conditions
5. **JSON policy**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "ec2:DescribeInstances",
           "ec2:StartInstances",
           "ec2:StopInstances"
         ],
         "Resource": "*",
         "Condition": {
           "StringEquals": {
             "ec2:ResourceTag/Environment": "development"
           }
         }
       }
     ]
   }
   ```

**Step 2: Create IAM Role**
1. **Roles** → "Create role"
2. **Trust policy**: EC2 service
3. **Permissions**: Attach custom policy
4. **Role name**: "EC2ManagementRole"

**Verification Checklist:**
- ✅ Custom policy with minimal permissions
- ✅ Role created with proper trust relationship
- ✅ Principle of least privilege applied

### Task 23: Lambda with API Gateway
**Create a REST API using API Gateway that triggers Lambda functions.**

**Console-Based Solution:**

**Step 1: Create Lambda Function**
1. **Lambda Console** → "Create function"
2. **Function name**: "api-handler"
3. **Runtime**: Python 3.12
4. **Code**:
   ```python
   import json
   
   def lambda_handler(event, context):
       return {
           'statusCode': 200,
           'headers': {
               'Content-Type': 'application/json',
               'Access-Control-Allow-Origin': '*'
           },
           'body': json.dumps({
               'message': 'Hello from Lambda!',
               'httpMethod': event['httpMethod'],
               'path': event['path']
           })
       }
   ```

**Step 2: Create API Gateway**
1. **API Gateway Console** → "Create API"
2. **REST API** → "Build"
3. **API name**: "lambda-api"
4. **Create resource**: /hello
5. **Create method**: GET
6. **Integration**: Lambda function
7. **Deploy API** to stage "prod"

**Verification Checklist:**
- ✅ Lambda function created and tested
- ✅ API Gateway configured with Lambda integration
- ✅ API deployed and accessible via HTTPS
- ✅ CORS configured if needed

### Task 24: DynamoDB Table
**Create a DynamoDB table with GSI and implement basic CRUD operations.**

**Console-Based Solution:**

**Step 1: Create DynamoDB Table**
1. **DynamoDB Console** → "Create table"
2. **Table details**:
   - **Table name**: "UserProfiles"
   - **Partition key**: "userId" (String)
   - **Sort key**: "profileType" (String)
3. **Global secondary indexes**:
   - **Index name**: "EmailIndex"
   - **Partition key**: "email" (String)
4. **Table settings**: On-demand billing mode

**Step 2: Configure Advanced Settings**
1. **Encryption**: AWS managed key
2. **Point-in-time recovery**: Enable
3. **DynamoDB Streams**: Enable

**Verification Checklist:**
- ✅ Table created with partition and sort keys
- ✅ Global Secondary Index configured
- ✅ Point-in-time recovery enabled
- ✅ Encryption configured

### Task 25: ECS Cluster
**Set up an ECS cluster and run a containerized application.**

**Console-Based Solution:**

**Step 1: Create ECS Cluster**
1. **ECS Console** → "Create cluster"
2. **Cluster configuration**:
   - **Cluster name**: "webapp-cluster"
   - **Infrastructure**: AWS Fargate
3. **Monitoring**: Enable Container Insights

**Step 2: Create Task Definition**
1. **Task definitions** → "Create new task definition"
2. **Task definition family**: "webapp-task"
3. **Infrastructure requirements**:
   - **Launch type**: AWS Fargate
   - **CPU**: 0.25 vCPU
   - **Memory**: 0.5 GB
4. **Container definitions**:
   - **Name**: "webapp"
   - **Image**: nginx:latest
   - **Port mappings**: 80

**Step 3: Create Service**
1. **Services** → "Create service"
2. **Service configuration**:
   - **Service name**: "webapp-service"
   - **Desired tasks**: 2
3. **Networking**: Configure VPC and security groups

**Verification Checklist:**
- ✅ ECS cluster created with Fargate
- ✅ Task definition configured properly
- ✅ Service running with desired task count
- ✅ Application accessible via load balancer

### Task 26: ECR Repository
**Create an ECR repository and push a Docker image to it.**

**Console-Based Solution:**

**Step 1: Create ECR Repository**
1. **ECR Console** → "Create repository"
2. **Repository configuration**:
   - **Repository name**: "webapp-repo"
   - **Tag immutability**: Enabled
   - **Scan on push**: Enabled
   - **Encryption**: AES-256

**Step 2: Push Docker Image**
1. Get login token:
   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
   ```
2. Build and tag image:
   ```bash
   docker build -t webapp-repo .
   docker tag webapp-repo:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/webapp-repo:latest
   ```
3. Push image:
   ```bash
   docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/webapp-repo:latest
   ```

**Verification Checklist:**
- ✅ ECR repository created with proper settings
- ✅ Docker image built and tagged
- ✅ Image pushed to ECR successfully
- ✅ Image scanning completed

### Task 27: Systems Manager
**Use SSM to manage EC2 instances without SSH access.**

**Console-Based Solution:**

**Step 1: Configure IAM Role for EC2**
1. Create IAM role with **AmazonSSMManagedInstanceCore** policy
2. Attach role to EC2 instances

**Step 2: Use Session Manager**
1. **Systems Manager Console** → "Session Manager"
2. **Start session** → Select instance
3. Execute commands through web-based shell

**Step 3: Use Run Command**
1. **Systems Manager Console** → "Run Command"
2. **Document**: AWS-RunShellScript
3. **Commands**:
   ```bash
   sudo yum update -y
   sudo systemctl status httpd
   ```
4. **Targets**: Select instances by tags

**Verification Checklist:**
- ✅ SSM Agent installed and running
- ✅ Session Manager access working
- ✅ Run Command executing successfully
- ✅ Parameter Store configured for secrets

### Task 28: CloudTrail Setup
**Configure CloudTrail for auditing and compliance.**

**Console-Based Solution:**

**Step 1: Create CloudTrail**
1. **CloudTrail Console** → "Create trail"
2. **Trail configuration**:
   - **Trail name**: "audit-trail"
   - **Storage location**: Create new S3 bucket
   - **Log file encryption**: Enable with KMS key
3. **Events**: Management and data events
4. **Insights**: Enable CloudTrail Insights

**Step 2: Configure Log Analysis**
1. **CloudWatch Logs**: Send logs to CloudWatch
2. **EventBridge**: Create rules for real-time monitoring
3. **Athena**: Query logs using SQL

**Verification Checklist:**
- ✅ CloudTrail capturing all API calls
- ✅ Logs encrypted and stored securely
- ✅ Real-time monitoring configured
- ✅ Log analysis tools set up

### Task 29: KMS Key Management
**Create and use KMS keys for encryption at rest.**

**Console-Based Solution:**

**Step 1: Create KMS Key**
1. **KMS Console** → "Create key"
2. **Key type**: Symmetric
3. **Key usage**: Encrypt and decrypt
4. **Key policy**: Define permissions
5. **Aliases**: Create meaningful alias

**Step 2: Use KMS Key**
1. **S3**: Configure bucket encryption with KMS key
2. **RDS**: Enable encryption with custom KMS key
3. **EBS**: Encrypt volumes with KMS key

**Verification Checklist:**
- ✅ KMS key created with proper permissions
- ✅ Services configured to use custom key
- ✅ Key rotation enabled
- ✅ Access logging configured

### Task 30: Secrets Manager
**Store and retrieve database credentials using AWS Secrets Manager.**

**Console-Based Solution:**

**Step 1: Store Secret**
1. **Secrets Manager Console** → "Store a new secret"
2. **Secret type**: Database credentials
3. **Database**: Select RDS instance
4. **Credentials**: Username and password
5. **Rotation**: Configure automatic rotation

**Step 2: Retrieve Secret in Application**
1. **Lambda function** code to retrieve secret:
   ```python
   import boto3
   import json
   
   def get_secret():
       session = boto3.session.Session()
       client = session.client('secretsmanager', region_name='us-east-1')
       secret_value = client.get_secret_value(SecretId='prod/db/credentials')
       return json.loads(secret_value['SecretString'])
   ```

**Verification Checklist:**
- ✅ Database credentials stored securely
- ✅ Automatic rotation configured
- ✅ Applications retrieving secrets programmatically
- ✅ Access logging enabled

## Advanced Level Solutions (Tasks 31-50)

### Task 31: ElastiCache Redis Cluster
**Set up Redis ElastiCache cluster for application caching with replication and failover.**

**Console-Based Solution:**

**Step 1: Navigate to ElastiCache Console**
1. Go to **Amazon ElastiCache Console**
2. Click "Redis clusters" in left navigation
3. Click "Create Redis cluster"

**Step 2: Configure Cluster Settings**
1. **Cluster creation method**: Easy create (or Configure and create for advanced)
2. **Cluster info**:
   - **Name**: "app-cache-cluster"
   - **Description**: "Redis cluster for application caching"
   - **Node type**: cache.t3.micro (or larger for production)
   - **Number of replicas**: 2 (for high availability)
3. **Connectivity**:
   - **Network type**: IPv4
   - **Subnet group**: Create new or select existing
   - **VPC**: Select your application VPC

**Step 3: Advanced Configuration**
1. **Multi-AZ**: Enable (for automatic failover)
2. **Backup**:
   - **Enable automatic backups**: Yes
   - **Backup retention period**: 7 days
   - **Backup window**: 03:00-05:00 UTC (low traffic period)
3. **Maintenance**:
   - **Maintenance window**: sun:05:00-sun:06:00 (weekly)
   - **Topic for SNS notification**: Select or create SNS topic

**Step 4: Security Configuration**
1. **Security groups**: Create or select security group
   - **Inbound rules**: Allow port 6379 from application security group
   - **Source**: Application servers security group ID
2. **Encryption**:
   - **Encryption at rest**: Enable
   - **Encryption in transit**: Enable
   - **Auth token**: Set strong password for Redis AUTH

**Step 5: Create Subnet Group (if needed)**
1. **Subnet groups** → "Create subnet group"
2. **Name**: "cache-subnet-group"
3. **VPC**: Select your VPC
4. **Subnets**: Select private subnets across multiple AZs
5. Click "Create"

**Step 6: Monitor Cluster Creation**
1. Click "Create cluster"
2. **Status progression**:
   - Creating → Available (typically 10-15 minutes)
3. **Endpoints**: Note primary and reader endpoints
4. **Configuration endpoint**: For cluster mode enabled

**Step 7: Configure Application Connection**
1. **Primary endpoint**: Use for write operations
2. **Reader endpoint**: Use for read operations
3. **Connection string example**:
   ```python
   import redis
   
   # Redis cluster configuration
   redis_client = redis.Redis(
       host='app-cache-cluster.abc123.cache.amazonaws.com',
       port=6379,
       password='your-auth-token',
       ssl=True,
       decode_responses=True
   )
   
   # Test connection
   redis_client.ping()
   ```

**Step 8: Implement Caching Strategy**
1. **Cache patterns**:
   - **Cache-aside**: Application manages cache
   - **Write-through**: Write to cache and database
   - **Write-behind**: Write to cache, async to database
2. **Example implementation**:
   ```python
   def get_user_profile(user_id):
       # Try cache first
       cache_key = f"user:profile:{user_id}"
       cached_data = redis_client.get(cache_key)
       
       if cached_data:
           return json.loads(cached_data)
       
       # Cache miss - get from database
       user_data = database.get_user(user_id)
       
       # Store in cache with TTL
       redis_client.setex(
           cache_key, 
           3600,  # 1 hour TTL
           json.dumps(user_data)
       )
       
       return user_data
   ```

**Step 9: Configure CloudWatch Monitoring**
1. **Metrics to monitor**:
   - **CPUUtilization**: Should be < 80%
   - **DatabaseMemoryUsagePercentage**: Monitor memory usage
   - **CacheMisses**: Track cache effectiveness
   - **ReplicationLag**: Monitor replica lag
2. **Create CloudWatch alarms**:
   - High CPU utilization (> 80%)
   - High memory usage (> 85%)
   - High cache miss ratio (> 20%)

**Step 10: Backup and Recovery Testing**
1. **Manual backup**:
   - Cluster actions → "Backup"
   - **Backup name**: "manual-backup-YYYY-MM-DD"
2. **Restore testing**:
   - Create new cluster from backup
   - Verify data integrity
   - Test application connectivity

**Verification Checklist:**
- ✅ Redis cluster created with replication
- ✅ Multi-AZ failover configured
- ✅ Security groups properly configured
- ✅ Encryption enabled for data at rest and in transit
- ✅ Application successfully connecting to cluster
- ✅ Caching strategy implemented
- ✅ CloudWatch monitoring configured
- ✅ Backup and recovery tested

### Task 32: SQS Queue System
**Create SQS queues for decoupling application components with dead letter queues and monitoring.**

**Console-Based Solution:**

**Step 1: Navigate to SQS Console**
1. Go to **Amazon SQS Console**
2. Click "Create queue"

**Step 2: Configure Main Queue**
1. **Queue type**: Standard
   - **Alternative**: FIFO (for ordered processing, lower throughput)
2. **Queue details**:
   - **Name**: "order-processing-queue"
   - **Visibility timeout**: 30 seconds (adjust based on processing time)
   - **Message retention period**: 14 days
   - **Delivery delay**: 0 seconds
   - **Receive message wait time**: 0 seconds (short polling)

**Step 3: Configure Dead Letter Queue**
1. First, create the DLQ:
   - **Name**: "order-processing-dlq"
   - **Type**: Standard (same as main queue)
2. **Dead letter queue settings** (in main queue):
   - **Enable**: Yes
   - **Queue**: Select "order-processing-dlq"
   - **Maximum receives**: 3 (retry attempts before DLQ)

**Step 4: Access Policy Configuration**
1. **Access policy**: Advanced
2. **JSON policy**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::ACCOUNT-ID:root"
         },
         "Action": [
           "sqs:SendMessage",
           "sqs:ReceiveMessage",
           "sqs:DeleteMessage",
           "sqs:GetQueueAttributes"
         ],
         "Resource": "arn:aws:sqs:REGION:ACCOUNT-ID:order-processing-queue"
       }
     ]
   }
   ```

**Step 5: Encryption Configuration**
1. **Server-side encryption**: Enable
2. **Encryption key**: AWS managed key (SQS)
   - **Alternative**: Customer managed KMS key for additional control
3. **Data key reuse period**: 300 seconds (default)

**Step 6: Tags and Additional Settings**
1. **Tags**:
   - Environment: production
   - Application: order-processing
   - Team: backend-services
2. **Redrive allow policy**: Configure which queues can send to DLQ

**Step 7: Create Producer Application**
1. **IAM role for producer**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "sqs:SendMessage",
           "sqs:GetQueueUrl"
         ],
         "Resource": "arn:aws:sqs:*:*:order-processing-queue"
       }
     ]
   }
   ```

2. **Producer code example**:
   ```python
   import boto3
   import json
   from datetime import datetime
   
   sqs = boto3.client('sqs')
   queue_url = 'https://sqs.us-east-1.amazonaws.com/123456789012/order-processing-queue'
   
   def send_order_message(order_data):
       message_body = json.dumps({
           'orderId': order_data['id'],
           'customerId': order_data['customer_id'],
           'items': order_data['items'],
           'timestamp': datetime.utcnow().isoformat(),
           'priority': order_data.get('priority', 'normal')
       })
       
       response = sqs.send_message(
           QueueUrl=queue_url,
           MessageBody=message_body,
           MessageAttributes={
               'Priority': {
                   'StringValue': order_data.get('priority', 'normal'),
                   'DataType': 'String'
               },
               'CustomerType': {
                   'StringValue': order_data.get('customer_type', 'regular'),
                   'DataType': 'String'
               }
           }
       )
       
       return response['MessageId']
   ```

**Step 8: Create Consumer Application**
1. **IAM role for consumer**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "sqs:ReceiveMessage",
           "sqs:DeleteMessage",
           "sqs:GetQueueAttributes"
         ],
         "Resource": "arn:aws:sqs:*:*:order-processing-queue"
       }
     ]
   }
   ```

2. **Consumer code example**:
   ```python
   import boto3
   import json
   import time
   
   sqs = boto3.client('sqs')
   queue_url = 'https://sqs.us-east-1.amazonaws.com/123456789012/order-processing-queue'
   
   def process_messages():
       while True:
           # Long polling (recommended)
           response = sqs.receive_message(
               QueueUrl=queue_url,
               MaxNumberOfMessages=10,
               WaitTimeSeconds=20,  # Long polling
               MessageAttributeNames=['All']
           )
           
           messages = response.get('Messages', [])
           
           for message in messages:
               try:
                   # Process the message
                   order_data = json.loads(message['Body'])
                   process_order(order_data)
                   
                   # Delete message after successful processing
                   sqs.delete_message(
                       QueueUrl=queue_url,
                       ReceiptHandle=message['ReceiptHandle']
                   )
                   
               except Exception as e:
                   print(f"Error processing message: {e}")
                   # Message will be returned to queue after visibility timeout
                   
           if not messages:
               time.sleep(5)  # Brief pause if no messages
   
   def process_order(order_data):
       # Your order processing logic here
       print(f"Processing order {order_data['orderId']}")
       # Simulate processing time
       time.sleep(2)
   ```

**Step 9: Configure CloudWatch Monitoring**
1. **Key metrics to monitor**:
   - **ApproximateNumberOfMessages**: Messages in queue
   - **ApproximateNumberOfMessagesVisible**: Available messages
   - **ApproximateNumberOfMessagesNotVisible**: Processing messages
   - **NumberOfMessagesSent**: Incoming message rate
   - **NumberOfMessagesReceived**: Processing rate
   - **NumberOfMessagesDeleted**: Successfully processed

2. **Create CloudWatch alarms**:
   - **Queue depth alarm**: > 1000 messages
   - **DLQ alarm**: > 0 messages in dead letter queue
   - **Processing lag**: Messages older than threshold

**Step 10: Implement Message Filtering**
1. **Create filtered queues for different priorities**:
   - **High priority queue**: "order-processing-high"
   - **Normal priority queue**: "order-processing-normal"

2. **Use SNS fanout pattern** (optional):
   - SNS topic receives all orders
   - Multiple SQS queues subscribe with filters
   - Different consumers for different priorities

**Step 11: Auto Scaling Integration**
1. **Configure Auto Scaling** based on queue depth:
   ```python
   # CloudWatch custom metric for scaling
   import boto3
   
   cloudwatch = boto3.client('cloudwatch')
   
   def publish_queue_depth_metric():
       sqs = boto3.client('sqs')
       
       response = sqs.get_queue_attributes(
           QueueUrl=queue_url,
           AttributeNames=['ApproximateNumberOfMessages']
       )
       
       queue_depth = int(response['Attributes']['ApproximateNumberOfMessages'])
       
       cloudwatch.put_metric_data(
           Namespace='SQS/Custom',
           MetricData=[
               {
                   'MetricName': 'QueueDepth',
                   'Value': queue_depth,
                   'Unit': 'Count'
               }
           ]
       )
   ```

**Step 12: Error Handling and Retry Logic**
1. **Exponential backoff** for temporary failures:
   ```python
   import time
   import random
   
   def process_with_retry(message, max_retries=3):
       for attempt in range(max_retries):
           try:
               process_order(json.loads(message['Body']))
               return True
           except Exception as e:
               if attempt < max_retries - 1:
                   delay = (2 ** attempt) + random.uniform(0, 1)
                   time.sleep(delay)
               else:
                   # Log error and let message go to DLQ
                   print(f"Failed after {max_retries} attempts: {e}")
                   return False
   ```

**Step 13: Cost Optimization**
1. **Use long polling** to reduce empty receive requests
2. **Batch operations** when possible:
   ```python
   # Send messages in batches
   def send_message_batch(messages):
       entries = []
       for i, msg in enumerate(messages):
           entries.append({
               'Id': str(i),
               'MessageBody': json.dumps(msg)
           })
       
       sqs.send_message_batch(
           QueueUrl=queue_url,
           Entries=entries
       )
   ```

3. **Monitor and optimize**:
   - Review message retention periods
   - Adjust visibility timeouts based on processing time
   - Use appropriate queue types (Standard vs FIFO)

**Verification Checklist:**
- ✅ Main queue created with appropriate settings
- ✅ Dead letter queue configured and working
- ✅ Producer application sending messages successfully
- ✅ Consumer application processing messages reliably
- ✅ CloudWatch monitoring and alarms configured
- ✅ Error handling and retry logic implemented
- ✅ Security policies properly configured
- ✅ Cost optimization measures applied

### Task 33: CodeCommit Repository
**Set up CodeCommit repository with branching strategy and collaboration workflows.**

**Console-Based Solution:**

**Step 1: Navigate to CodeCommit Console**
1. Go to **AWS CodeCommit Console**
2. Click "Create repository"

**Step 2: Configure Repository**
1. **Repository settings**:
   - **Repository name**: "webapp-source"
   - **Description**: "Main source code repository for web application"
   - **Tags**:
     - Project: WebApp
     - Team: Development
     - Environment: All

**Step 3: Set Up IAM Users and Permissions**
1. **Create IAM group** for developers:
   - **Group name**: "CodeCommit-Developers"
   - **Policies**: AWSCodeCommitPowerUser

2. **Create IAM policy** for fine-grained access:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "codecommit:BatchGet*",
           "codecommit:Get*",
           "codecommit:Describe*",
           "codecommit:List*",
           "codecommit:GitPull"
         ],
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "codecommit:GitPush",
           "codecommit:MergeBranchesByFastForward",
           "codecommit:MergeBranchesBySquash",
           "codecommit:MergeBranchesByThreeWay",
           "codecommit:MergePullRequestByFastForward",
           "codecommit:MergePullRequestBySquash",
           "codecommit:MergePullRequestByThreeWay"
         ],
         "Resource": "arn:aws:codecommit:*:*:webapp-source",
         "Condition": {
           "StringNotEquals": {
             "codecommit:References": [
               "refs/heads/main",
               "refs/heads/production"
             ]
           }
         }
       }
     ]
   }
   ```

**Step 4: Configure Git Credentials**
1. **For IAM users**:
   - Go to IAM Console → Users → Select user
   - **Security credentials** tab
   - **HTTPS Git credentials for AWS CodeCommit** → Generate credentials
   - Download and securely store credentials

2. **For federated users** (recommended):
   - Install **git-remote-codecommit**:
     ```bash
     pip install git-remote-codecommit
     ```
   - Configure AWS CLI with appropriate profile

**Step 5: Clone Repository Locally**
1. **Using HTTPS with IAM credentials**:
   ```bash
   git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/webapp-source
   cd webapp-source
   ```

2. **Using git-remote-codecommit** (recommended):
   ```bash
   git clone codecommit://MyProfile@webapp-source
   cd webapp-source
   ```

**Step 6: Set Up Initial Project Structure**
1. **Create initial files**:
   ```bash
   # Create basic project structure
   mkdir -p src/{components,services,utils}
   mkdir -p tests/{unit,integration}
   mkdir -p docs/{api,deployment}
   mkdir -p .github/workflows
   
   # Create initial files
   cat > README.md << 'EOF'
   # Web Application
   
   ## Project Description
   Modern web application with React frontend and Node.js backend.
   
   ## Getting Started
   1. Clone the repository
   2. Install dependencies: `npm install`
   3. Start development server: `npm start`
   
   ## Branching Strategy
   - `main`: Production-ready code
   - `develop`: Integration branch for features
   - `feature/*`: Feature development branches
   - `hotfix/*`: Emergency fixes for production
   
   ## Contributing
   1. Create feature branch from develop
   2. Make changes and commit
   3. Create pull request to develop
   4. Code review and merge
   EOF
   
   cat > .gitignore << 'EOF'
   # Dependencies
   node_modules/
   npm-debug.log*
   
   # Production builds
   build/
   dist/
   
   # Environment variables
   .env
   .env.local
   .env.development.local
   .env.test.local
   .env.production.local
   
   # IDE files
   .vscode/
   .idea/
   *.swp
   *.swo
   
   # OS files
   .DS_Store
   Thumbs.db
   EOF
   
   cat > package.json << 'EOF'
   {
     "name": "webapp-source",
     "version": "1.0.0",
     "description": "Web application source code",
     "main": "src/index.js",
     "scripts": {
       "start": "node src/index.js",
       "dev": "nodemon src/index.js",
       "test": "jest",
       "build": "webpack --mode production",
       "lint": "eslint src/"
     },
     "dependencies": {
       "express": "^4.18.0",
       "react": "^18.2.0",
       "react-dom": "^18.2.0"
     },
     "devDependencies": {
       "jest": "^29.0.0",
       "eslint": "^8.0.0",
       "nodemon": "^2.0.0",
       "webpack": "^5.0.0"
     }
   }
   EOF
   ```

**Step 7: Establish Branching Strategy**
1. **Create and push initial commit**:
   ```bash
   git add .
   git commit -m "Initial project setup with basic structure"
   git push origin main
   ```

2. **Create develop branch**:
   ```bash
   git checkout -b develop
   git push origin develop
   ```

3. **Set up branch protection** (via AWS Console):
   - Repository → **Settings** → **Branch rules**
   - **Rule name**: "main-protection"
   - **Branch pattern**: "main"
   - **Restrictions**:
     - ✅ Require pull request reviews
     - ✅ Require status checks to pass
     - ✅ Restrict push to administrators only

**Step 8: Configure Pull Request Templates**
1. **Create pull request template**:
   ```bash
   mkdir -p .github/pull_request_template
   
   cat > .github/pull_request_template/feature.md << 'EOF'
   ## Feature Description
   Brief description of the feature/change.
   
   ## Type of Change
   - [ ] Bug fix (non-breaking change that fixes an issue)
   - [ ] New feature (non-breaking change that adds functionality)
   - [ ] Breaking change (fix or feature that causes existing functionality to not work as expected)
   - [ ] Documentation update
   
   ## Testing
   - [ ] Unit tests pass
   - [ ] Integration tests pass
   - [ ] Manual testing completed
   
   ## Checklist
   - [ ] Code follows style guidelines
   - [ ] Self-review completed
   - [ ] Code is commented where necessary
   - [ ] Documentation updated
   - [ ] No sensitive data included
   
   ## Related Issues
   Closes #(issue number)
   EOF
   ```

**Step 9: Set Up Approval Rule Templates**
1. **Navigate to CodeCommit Console**
2. **Repository** → **Settings** → **Approval rule templates**
3. **Create template**:
   - **Template name**: "Require-Two-Approvals"
   - **Description**: "Require two approvals for main branch"
   - **Content**:
     ```json
     {
       "Version": "2018-11-08",
       "DestinationReferences": ["refs/heads/main"],
       "Statements": [
         {
           "Type": "Approvers",
           "NumberOfApprovalsNeeded": 2,
           "ApprovalPoolMembers": [
             "arn:aws:sts::ACCOUNT-ID:assumed-role/CodeCommit-Senior-Developers/*"
           ]
         }
       ]
     }
     ```

**Step 10: Integrate with Notifications**
1. **Create SNS topic** for CodeCommit notifications:
   - **Topic name**: "codecommit-notifications"
   - **Subscriptions**: Team email addresses

2. **Configure notification rules**:
   - Repository → **Settings** → **Notification rules**
   - **Rule name**: "PR-and-Push-Notifications"
   - **Events**:
     - Pull request created
     - Pull request source updated
     - Pull request status changed
     - Comments on pull requests
   - **Target**: SNS topic

**Step 11: Set Up Webhooks for CI/CD**
1. **Repository triggers**:
   - Repository → **Settings** → **Triggers**
   - **Trigger name**: "Build-Trigger"
   - **Events**: Push to existing branch
   - **Branches**: main, develop
   - **Service**: AWS Lambda or Amazon SNS
   - **Service endpoint**: CodeBuild webhook or Lambda function

**Step 12: Implement Commit Message Standards**
1. **Create commit message template**:
   ```bash
   cat > .gitmessage << 'EOF'
   # Type: Subject (50 chars max)
   #
   # Body: Explain what and why (wrap at 72 chars)
   #
   # Types:
   # feat: A new feature
   # fix: A bug fix
   # docs: Documentation only changes
   # style: Changes that do not affect the meaning of the code
   # refactor: A code change that neither fixes a bug nor adds a feature
   # perf: A code change that improves performance
   # test: Adding missing tests or correcting existing tests
   # chore: Changes to the build process or auxiliary tools
   #
   # Example:
   # feat: Add user authentication system
   #
   # Implemented OAuth 2.0 integration with Google and GitHub
   # providers. Added JWT token management and refresh logic.
   # Includes comprehensive unit tests and documentation.
   #
   # Closes #123
   EOF
   
   # Configure git to use the template
   git config commit.template .gitmessage
   ```

**Step 13: Monitor Repository Activity**
1. **CloudWatch metrics** for CodeCommit:
   - Number of repositories
   - Number of pull requests
   - Repository size

2. **Custom monitoring** with CloudTrail:
   - Track all API calls to CodeCommit
   - Monitor for unauthorized access attempts
   - Audit code changes and access patterns

**Step 14: Backup and Migration Strategy**
1. **Regular repository backups**:
   - Use AWS Backup service or custom scripts
   - Mirror to secondary Git repository
   - Export repository data periodically

2. **Migration preparation**:
   - Document all repository settings
   - Export branch protection rules
   - Backup approval rule templates
   - List all access permissions

**Verification Checklist:**
- ✅ Repository created with proper permissions
- ✅ Branching strategy implemented (main, develop, feature branches)
- ✅ Branch protection rules configured
- ✅ Pull request workflows established
- ✅ Approval rule templates created
- ✅ Notification system configured
- ✅ Git credentials and access properly set up
- ✅ Commit message standards documented
- ✅ Integration with CI/CD pipeline prepared
- ✅ Monitoring and auditing configured

### Task 34: CodeBuild Project
**Create a comprehensive CodeBuild project with multiple build environments and test stages.**

**Console-Based Solution:**

**Step 1: Navigate to CodeBuild Console**
1. Go to **AWS CodeBuild Console**
2. Click "Create build project"

**Step 2: Configure Project Settings**
1. **Project configuration**:
   - **Project name**: "webapp-build-pipeline"
   - **Description**: "Complete build pipeline for web application with testing and artifact generation"
   - **Tags**:
     - Environment: development
     - Application: webapp
     - Team: devops

**Step 3: Configure Source**
1. **Source provider**: AWS CodeCommit
2. **Repository**: webapp-source (from previous task)
3. **Reference type**: Branch
4. **Branch**: develop
5. **Git clone depth**: Full clone (for complete history)
6. **Git submodules**: Include (if using submodules)

**Step 4: Configure Environment**
1. **Environment image**: Managed image
2. **Operating system**: Ubuntu
3. **Runtime(s)**: Standard
4. **Image**: aws/codebuild/standard:7.0 (latest)
5. **Image version**: Always use the latest image
6. **Environment type**: Linux
7. **Privileged**: Enable (required for Docker builds)
8. **Service role**: 
   - **New service role**: Create new role
   - **Role name**: codebuild-webapp-service-role

**Step 5: Configure Additional Settings**
1. **Timeout**: 60 minutes (adjust based on build complexity)
2. **Queued timeout**: 8 hours
3. **Certificate**: Use default
4. **Compute**: 3 GB memory, 2 vCPUs (adjust based on needs)
5. **Environment variables**:
   - **Name**: NODE_ENV, **Value**: development
   - **Name**: AWS_DEFAULT_REGION, **Value**: us-east-1
   - **Name**: ARTIFACT_BUCKET, **Value**: webapp-build-artifacts

**Step 6: Create Buildspec File**
1. **Buildspec**: Use a buildspec file
2. **Buildspec name**: buildspec.yml (in source root)

3. **Create comprehensive buildspec.yml**:
   ```yaml
   version: 0.2
   
   env:
     variables:
       NODE_ENV: "development"
       CACHE_CONTROL: "max-age=86400"
     parameter-store:
       DATABASE_URL: "/webapp/dev/database-url"
       API_KEY: "/webapp/dev/api-key"
   
   phases:
     install:
       runtime-versions:
         nodejs: 18
         python: 3.11
       commands:
         - echo Logging in to Amazon ECR...
         - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
         - echo Installing dependencies...
         - npm install
         - pip install awscli pytest
         
     pre_build:
       commands:
         - echo Pre-build started on `date`
         - echo Running tests...
         - npm run lint
         - npm run test:unit
         - npm run test:integration
         - echo Running security scan...
         - npm audit --audit-level high
         - echo Building Docker image...
         - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
         - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
         
     build:
       commands:
         - echo Build started on `date`
         - echo Building the application...
         - npm run build
         - echo Build completed on `date`
         - echo Pushing Docker image...
         - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
         
     post_build:
       commands:
         - echo Post-build started on `date`
         - echo Running post-build tests...
         - npm run test:e2e
         - echo Generating documentation...
         - npm run docs:generate
         - echo Creating deployment package...
         - aws s3 cp ./build s3://$ARTIFACT_BUCKET/builds/$CODEBUILD_BUILD_NUMBER/ --recursive
         - echo Build completed on `date`
   
   artifacts:
     files:
       - '**/*'
     base-directory: build
     name: webapp-build-$CODEBUILD_BUILD_NUMBER
   
   cache:
     paths:
       - node_modules/**/*
       - .npm/**/*
   
   reports:
     test_reports:
       files:
         - 'test-results.xml'
       base-directory: 'test-reports'
       file-format: 'JUNITXML'
     coverage_reports:
       files:
         - 'clover.xml'
       base-directory: 'coverage'
       file-format: 'CLOVERXML'
   ```

**Step 7: Configure Artifacts**
1. **Type**: Amazon S3
2. **Bucket name**: webapp-build-artifacts (create if needed)
3. **Name**: webapp-build-$CODEBUILD_BUILD_NUMBER
4. **Path**: builds/
5. **Namespace type**: Build ID
6. **Packaging**: Zip
7. **Encryption**: Enable with AWS KMS key

**Step 8: Configure Logs**
1. **CloudWatch Logs**: Enable
   - **Group name**: /aws/codebuild/webapp-build-pipeline
   - **Stream name**: Leave blank (auto-generated)
2. **S3 logs**: Enable (optional)
   - **Bucket**: webapp-build-logs
   - **Path prefix**: build-logs/

**Step 9: Create IAM Service Role Permissions**
1. **Attach additional policies** to CodeBuild service role:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "logs:CreateLogGroup",
           "logs:CreateLogStream",
           "logs:PutLogEvents"
         ],
         "Resource": "arn:aws:logs:*:*:*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "s3:GetObject",
           "s3:GetObjectVersion",
           "s3:PutObject"
         ],
         "Resource": [
           "arn:aws:s3:::webapp-build-artifacts/*",
           "arn:aws:s3:::webapp-build-logs/*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "ecr:BatchCheckLayerAvailability",
           "ecr:GetDownloadUrlForLayer",
           "ecr:BatchGetImage",
           "ecr:GetAuthorizationToken"
         ],
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "ssm:GetParameters",
           "ssm:GetParameter"
         ],
         "Resource": "arn:aws:ssm:*:*:parameter/webapp/*"
       }
     ]
   }
   ```

**Step 10: Set Up Build Triggers**
1. **Webhook**: Enable for automatic builds
2. **Events**: 
   - PUSH (for commits to develop branch)
   - PULL_REQUEST_CREATED
   - PULL_REQUEST_UPDATED
3. **Filter groups**:
   - **Event**: PUSH
   - **Branch**: ^refs/heads/develop$
   - **Event**: PULL_REQUEST_CREATED, PULL_REQUEST_UPDATED
   - **Base branch**: ^refs/heads/main$

**Step 11: Configure Multiple Build Projects**
1. **Create additional build projects** for different environments:

   **Production Build Project**:
   - **Name**: webapp-build-production
   - **Source branch**: main
   - **Environment variables**: NODE_ENV=production
   - **Additional security scanning**
   - **Performance testing**

   **Feature Branch Testing**:
   - **Name**: webapp-test-features
   - **Source**: All branches except main/develop
   - **Lightweight testing only**
   - **No deployment artifacts**

**Step 12: Implement Build Caching**
1. **Cache configuration**:
   - **Type**: S3
   - **Location**: webapp-build-cache/node_modules
2. **Local caching** in buildspec:
   ```yaml
   cache:
     paths:
       - node_modules/**/*
       - .npm/**/*
       - $HOME/.cache/pip/**/*
   ```

**Step 13: Set Up Build Notifications**
1. **Create SNS topic**: "codebuild-notifications"
2. **Configure CloudWatch Events rule**:
   - **Event pattern**:
     ```json
     {
       "source": ["aws.codebuild"],
       "detail-type": ["CodeBuild Build State Change"],
       "detail": {
         "build-status": ["FAILED", "SUCCEEDED"],
         "project-name": ["webapp-build-pipeline"]
       }
     }
     ```
   - **Target**: SNS topic

**Step 14: Implement Build Metrics and Monitoring**
1. **CloudWatch dashboards** for build metrics:
   - Build success rate
   - Build duration trends
   - Failed builds by reason
   - Resource utilization

2. **Custom metrics** in buildspec:
   ```yaml
   post_build:
     commands:
       - |
         aws cloudwatch put-metric-data \
           --namespace "CodeBuild/Custom" \
           --metric-data MetricName=BuildDuration,Value=$BUILD_DURATION,Unit=Seconds \
           --region $AWS_DEFAULT_REGION
   ```

**Step 15: Security and Compliance**
1. **Enable VPC configuration** if needed:
   - **VPC**: Select application VPC
   - **Subnets**: Private subnets
   - **Security groups**: Allow outbound HTTPS

2. **Secrets management**:
   - Store sensitive data in Systems Manager Parameter Store
   - Use IAM roles instead of hardcoded credentials
   - Scan for secrets in code before build

**Step 16: Test and Validate Build**
1. **Start manual build**:
   - Project → "Start build"
   - **Source version**: develop
   - **Environment variables override**: Add test-specific variables

2. **Monitor build progress**:
   - **Build logs**: Real-time log streaming
   - **Build phases**: Track phase execution
   - **Artifacts**: Verify artifact generation

**Verification Checklist:**
- ✅ CodeBuild project created with proper configuration
- ✅ Comprehensive buildspec.yml with all build phases
- ✅ Automated testing integrated (unit, integration, e2e)
- ✅ Artifact generation and storage configured
- ✅ Docker image building and pushing working
- ✅ Build caching implemented for performance
- ✅ Webhook triggers configured for automatic builds
- ✅ Build notifications and monitoring set up
- ✅ Multiple build environments (dev, prod) configured
- ✅ Security scanning and compliance checks included
- ✅ IAM permissions properly configured
- ✅ Build logs and reports properly stored

### Task 35: CodePipeline
**Set up a complete CI/CD pipeline with multiple stages and approval processes.**

**Console-Based Solution:**

**Step 1: Navigate to CodePipeline Console**
1. Go to **AWS CodePipeline Console**
2. Click "Create pipeline"

**Step 2: Pipeline Settings**
1. **Pipeline name**: "webapp-deployment-pipeline"
2. **Service role**: 
   - **New service role**: Create new role
   - **Role name**: AWSCodePipelineServiceRole-webapp-pipeline
3. **Artifact store**:
   - **Type**: Amazon S3
   - **Bucket**: webapp-pipeline-artifacts (create new)
   - **Encryption**: AWS KMS (create new key or use existing)
4. **Advanced settings**:
   - **Variable namespace**: Leave default

**Step 3: Add Source Stage**
1. **Source provider**: AWS CodeCommit
2. **Repository name**: webapp-source
3. **Branch name**: main
4. **Detection options**: Amazon CloudWatch Events (recommended)
5. **Output artifacts**: SourceOutput

**Step 4: Add Build Stage**
1. **Add stage**: Build
2. **Add action group**:
   - **Action name**: Build-Application
   - **Action provider**: AWS CodeBuild
   - **Input artifacts**: SourceOutput
   - **Project name**: webapp-build-pipeline (from previous task)
   - **Output artifacts**: BuildOutput

**Step 5: Add Testing Stage**
1. **Add stage**: Test
2. **Action group 1 - Unit Tests**:
   - **Action name**: Unit-Tests
   - **Action provider**: AWS CodeBuild
   - **Project name**: Create new project "webapp-unit-tests"
   - **Input artifacts**: SourceOutput
   - **Output artifacts**: UnitTestOutput

3. **Action group 2 - Security Scan**:
   - **Action name**: Security-Scan
   - **Action provider**: AWS CodeBuild
   - **Project name**: Create new project "webapp-security-scan"
   - **Input artifacts**: SourceOutput
   - **Run order**: 1 (parallel with unit tests)

**Step 6: Add Manual Approval Stage**
1. **Add stage**: Manual-Approval
2. **Add action group**:
   - **Action name**: Review-Deployment
   - **Action provider**: Manual approval
   - **SNS topic ARN**: arn:aws:sns:region:account:deployment-approvals
   - **URL for review**: https://webapp-staging.example.com
   - **Comments**: "Please review the staging deployment and approve for production"

**Step 7: Add Staging Deployment Stage**
1. **Add stage**: Deploy-Staging
2. **Add action group**:
   - **Action name**: Deploy-To-Staging
   - **Action provider**: AWS CodeDeploy
   - **Input artifacts**: BuildOutput
   - **Application name**: webapp-staging
   - **Deployment group**: staging-servers

**Step 8: Add Production Deployment Stage**
1. **Add stage**: Deploy-Production
2. **Action group 1 - Blue/Green Deployment**:
   - **Action name**: Deploy-Blue-Green
   - **Action provider**: AWS CodeDeploy
   - **Input artifacts**: BuildOutput
   - **Application name**: webapp-production
   - **Deployment group**: production-servers

3. **Action group 2 - Update Lambda Functions**:
   - **Action name**: Update-Lambda
   - **Action provider**: AWS Lambda
   - **Function name**: update-production-config
   - **Input artifacts**: BuildOutput
   - **Run order**: 2 (after main deployment)

**Step 9: Create Supporting Build Projects**

**Unit Tests Project (webapp-unit-tests)**:
```yaml
# buildspec-unit-tests.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install
      
  pre_build:
    commands:
      - echo Starting unit tests...
      
  build:
    commands:
      - npm run test:unit -- --coverage --ci
      - npm run test:lint
      
  post_build:
    commands:
      - echo Unit tests completed
      
reports:
  unit_test_reports:
    files:
      - 'junit.xml'
    base-directory: 'test-reports'
    file-format: 'JUNITXML'
  coverage_reports:
    files:
      - 'clover.xml'
    base-directory: 'coverage'
    file-format: 'CLOVERXML'
```

**Security Scan Project (webapp-security-scan)**:
```yaml
# buildspec-security.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install
      - npm install -g npm-audit-ci-wrapper
      
  pre_build:
    commands:
      - echo Starting security scans...
      
  build:
    commands:
      - echo Running dependency vulnerability scan...
      - npm audit --audit-level high
      - echo Running SAST scan...
      - npm run security:sast
      - echo Running container image scan...
      - trivy image webapp:latest
      
  post_build:
    commands:
      - echo Security scans completed
      
reports:
  security_reports:
    files:
      - 'security-report.json'
    base-directory: 'security-reports'
    file-format: 'CUCUMBERJSON'
```

**Step 10: Configure Pipeline Variables**
1. **Pipeline variables**:
   - **DEPLOY_ENV**: staging
   - **VERSION**: #{codepipeline.PipelineExecutionId}
   - **REGION**: us-east-1

2. **Use variables in actions**:
   - In CodeBuild environment variables
   - In deployment configuration
   - In notification messages

**Step 11: Set Up CodeDeploy Applications**

**Staging Application**:
1. **Go to CodeDeploy Console** → "Create application"
2. **Application name**: webapp-staging
3. **Compute platform**: EC2/On-premises
4. **Deployment group**:
   - **Name**: staging-servers
   - **Service role**: CodeDeploy service role
   - **Auto Scaling groups**: staging-asg
   - **Deployment configuration**: CodeDeployDefault.EC2OneAtATime

**Production Application**:
1. **Application name**: webapp-production
2. **Deployment group**:
   - **Name**: production-servers
   - **Deployment configuration**: CodeDeployDefault.EC2BlueGreen
   - **Load balancer**: production-alb
   - **Auto Scaling groups**: production-asg

**Step 12: Create Deployment Scripts**
1. **appspec.yml** for CodeDeploy:
   ```yaml
   version: 0.0
   os: linux
   
   files:
     - source: /
       destination: /var/www/html
       
   hooks:
     BeforeInstall:
       - location: scripts/install_dependencies.sh
         timeout: 300
         runas: root
     ApplicationStart:
       - location: scripts/start_server.sh
         timeout: 300
         runas: root
     ApplicationStop:
       - location: scripts/stop_server.sh
         timeout: 300
         runas: root
     ValidateService:
       - location: scripts/validate_service.sh
         timeout: 300
   ```

2. **Deployment scripts**:
   ```bash
   # scripts/install_dependencies.sh
   #!/bin/bash
   yum update -y
   yum install -y nodejs npm
   cd /var/www/html
   npm install --production
   
   # scripts/start_server.sh
   #!/bin/bash
   cd /var/www/html
   npm start
   
   # scripts/validate_service.sh
   #!/bin/bash
   curl -f http://localhost:3000/health || exit 1
   ```

**Step 13: Configure Pipeline Notifications**
1. **Create notification rule**:
   - **Rule name**: webapp-pipeline-notifications
   - **Events**:
     - Pipeline execution started
     - Pipeline execution succeeded  
     - Pipeline execution failed
     - Stage execution failed
     - Manual approval needed
   - **Targets**: SNS topic, Slack webhook

2. **Create CloudWatch alarms**:
   - Pipeline failure rate
   - Average pipeline duration
   - Manual approval timeout

**Step 14: Implement Advanced Pipeline Features**

**Parallel Execution**:
```yaml
# Multiple actions in same stage
Deploy-Multi-Region:
  Actions:
    - Name: Deploy-US-East
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CodeDeploy
      RunOrder: 1
      
    - Name: Deploy-US-West
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CodeDeploy
      RunOrder: 1  # Same run order = parallel
```

**Conditional Execution**:
- Use pipeline execution conditions
- Configure action-level conditions based on variables
- Implement custom Lambda functions for complex logic

**Step 15: Set Up Pipeline Rollback Strategy**
1. **Automatic rollback triggers**:
   - CloudWatch alarms (error rate, latency)
   - Health check failures
   - Manual trigger

2. **Rollback Lambda function**:
   ```python
   import boto3
   
   def lambda_handler(event, context):
       codedeploy = boto3.client('codedeploy')
       
       # Stop current deployment
       response = codedeploy.stop_deployment(
           deploymentId=event['deploymentId'],
           autoRollbackEnabled=True
       )
       
       # Trigger previous version deployment
       codedeploy.create_deployment(
           applicationName='webapp-production',
           deploymentGroupName='production-servers',
           revision={
               'revisionType': 'S3',
               's3Location': {
                   'bucket': 'webapp-pipeline-artifacts',
                   'key': event['previousVersionKey']
               }
           },
           deploymentConfigName='CodeDeployDefault.EC2BlueGreenRollback'
       )
   ```

**Step 16: Pipeline Security Best Practices**
1. **IAM roles and policies**:
   - Least privilege access
   - Cross-account deployment roles
   - Resource-based policies

2. **Artifact encryption**:
   - KMS encryption for S3 artifacts
   - Encryption in transit
   - Secure parameter passing

3. **Audit and compliance**:
   - CloudTrail logging
   - Pipeline execution history
   - Approval audit trail

**Step 17: Performance Optimization**
1. **Caching strategies**:
   - Build artifact caching
   - Dependency caching
   - Docker layer caching

2. **Parallel execution**:
   - Independent stage parallelization
   - Multi-region deployments
   - Test parallelization

**Step 18: Monitoring and Alerting**
1. **Pipeline metrics dashboard**:
   - Success rate by stage
   - Average execution time
   - Failure reasons analysis
   - Deployment frequency

2. **Custom metrics**:
   ```python
   # In Lambda function or CodeBuild
   import boto3
   
   cloudwatch = boto3.client('cloudwatch')
   
   cloudwatch.put_metric_data(
       Namespace='CodePipeline/Custom',
       MetricData=[
           {
               'MetricName': 'DeploymentFrequency',
               'Value': 1,
               'Unit': 'Count',
               'Dimensions': [
                   {
                       'Name': 'Pipeline',
                       'Value': 'webapp-deployment-pipeline'
                   }
               ]
           }
       ]
   )
   ```

**Verification Checklist:**
- ✅ Complete CI/CD pipeline with source, build, test, and deploy stages
- ✅ Manual approval process configured with notifications
- ✅ Blue/green deployment strategy implemented
- ✅ Multiple environment deployments (staging, production)
- ✅ Automated testing integrated at multiple stages
- ✅ Security scanning included in pipeline
- ✅ Rollback strategy and procedures defined
- ✅ Pipeline notifications and monitoring configured
- ✅ Parallel execution for improved performance
- ✅ Proper IAM roles and security policies
- ✅ Artifact encryption and secure handling
- ✅ Pipeline variables and environment management

### Task 39: Transit Gateway
**Set up Transit Gateway to connect multiple VPCs with advanced routing and monitoring.**

**Console-Based Solution:**

**Step 1: Navigate to Transit Gateway Console**
1. Go to **VPC Console** → "Transit Gateways"
2. Click "Create Transit Gateway"

**Step 2: Configure Transit Gateway**
1. **Transit Gateway settings**:
   - **Name**: "central-tgw"
   - **Description**: "Central transit gateway for multi-VPC connectivity"
   - **Amazon side ASN**: 64512 (or custom ASN)
   - **CIDR blocks**: 10.99.0.0/16 (for TGW internal routing)
   - **DNS support**: Enable
   - **Multicast support**: Disable (unless needed)
   - **Default route table association**: Enable
   - **Default route table propagation**: Enable

2. **Advanced settings**:
   - **Auto accept shared attachments**: Disable (for security)
   - **Default route table association**: Enable
   - **Default route table propagation**: Enable

**Step 3: Create VPC Attachments**
1. **Production VPC attachment**:
   - **Name**: "prod-vpc-attachment"
   - **VPC**: Select production VPC
   - **Subnets**: Select one subnet per AZ (typically private)
   - **DNS support**: Enable
   - **IPv6 support**: Disable (unless using IPv6)

2. **Development VPC attachment**:
   - **Name**: "dev-vpc-attachment"
   - **VPC**: Select development VPC
   - **Subnets**: Select appropriate subnets

3. **Shared services VPC attachment**:
   - **Name**: "shared-services-attachment"
   - **VPC**: Select shared services VPC (AD, DNS, monitoring)
   - **Subnets**: Select subnets

**Step 4: Configure Custom Route Tables**
1. **Create security-focused route tables**:
   - **Production route table**: "prod-tgw-rt"
   - **Development route table**: "dev-tgw-rt"
   - **Shared services route table**: "shared-tgw-rt"

2. **Production route table configuration**:
   - **Associations**: Production VPC attachment
   - **Propagations**: Shared services VPC only
   - **Routes**:
     - 10.1.0.0/16 → Shared services attachment
     - 192.168.0.0/16 → VPN attachment (if applicable)

3. **Development route table configuration**:
   - **Associations**: Development VPC attachment
   - **Propagations**: Shared services VPC
   - **Routes**:
     - 10.1.0.0/16 → Shared services attachment
     - No direct route to production (security isolation)

**Step 5: Implement Network Segmentation**
1. **Security groups for TGW attachments**:
   ```json
   {
     "GroupName": "tgw-prod-sg",
     "Description": "Security group for production TGW traffic",
     "SecurityGroupRules": [
       {
         "IpProtocol": "tcp",
         "FromPort": 443,
         "ToPort": 443,
         "CidrIp": "10.1.0.0/16",
         "Description": "HTTPS from shared services"
       },
       {
         "IpProtocol": "tcp",
         "FromPort": 53,
         "ToPort": 53,
         "CidrIp": "10.1.0.0/16",
         "Description": "DNS from shared services"
       }
     ]
   }
   ```

2. **Network ACLs for additional security**:
   - Create specific NACLs for TGW traffic
   - Allow only necessary protocols and ports
   - Implement deny rules for known threats

**Step 6: Set Up VPN Connection via Transit Gateway**
1. **Create Customer Gateway**:
   - **Name**: "main-office-cgw"
   - **IP address**: Your public IP
   - **BGP ASN**: 65000 (or your ASN)
   - **Device**: Your VPN device model

2. **Create VPN attachment**:
   - **Type**: VPN
   - **Customer gateway**: Select created gateway
   - **Routing options**: Dynamic (BGP)
   - **Local IPv4 network CIDR**: 192.168.0.0/16
   - **Remote IPv4 network CIDR**: 10.0.0.0/8

3. **Configure on-premises routing**:
   - **Route table updates**: Add routes for AWS CIDRs
   - **BGP configuration**: Ensure proper route advertisement
   - **Firewall rules**: Allow TGW traffic

**Step 7: Implement Cross-Region Peering**
1. **Create Transit Gateway in secondary region**:
   - **Name**: "west-coast-tgw"
   - **Region**: us-west-2
   - **Configuration**: Similar to primary TGW

2. **Create TGW peering attachment**:
   - **Peering attachment name**: "east-west-peering"
   - **Transit Gateway**: Select local TGW
   - **Account**: Same account (or cross-account)
   - **Region**: us-west-2
   - **Transit Gateway ID**: Remote TGW ID

3. **Accept peering connection**:
   - **Switch to secondary region**
   - **Accept peering attachment**
   - **Configure route tables** in both regions

**Step 8: Monitor Transit Gateway Performance**
1. **CloudWatch metrics**:
   - **BytesIn/BytesOut**: Data transfer volume
   - **PacketDropCount**: Dropped packets
   - **PacketsIn/PacketsOut**: Packet volume
   - **AttachmentBandwidthUtilization**: Bandwidth usage

2. **Create CloudWatch dashboard**:
   ```json
   {
     "widgets": [
       {
         "type": "metric",
         "properties": {
           "metrics": [
             ["AWS/TransitGateway", "BytesIn", "TransitGateway", "tgw-xxxxx"],
             [".", "BytesOut", ".", "."]
           ],
           "period": 300,
           "stat": "Sum",
           "region": "us-east-1",
           "title": "TGW Data Transfer"
         }
       }
     ]
   }
   ```

**Step 9: Implement Route Automation**
1. **Lambda function for dynamic routing**:
   ```python
   import boto3
   import json
   
   def lambda_handler(event, context):
       """
       Automatically update TGW routes based on VPC changes
       """
       ec2 = boto3.client('ec2')
       
       # Get TGW route table
       tgw_rt_id = 'tgw-rtb-xxxxx'
       
       try:
           # Add route for new VPC
           if event['detail']['eventName'] == 'CreateVpcPeeringConnection':
               create_tgw_route(ec2, tgw_rt_id, event)
           
           # Remove route for deleted VPC
           elif event['detail']['eventName'] == 'DeleteVpcPeeringConnection':
               delete_tgw_route(ec2, tgw_rt_id, event)
               
       except Exception as e:
           print(f"Error updating TGW routes: {e}")
           raise e
   
   def create_tgw_route(ec2_client, route_table_id, event):
       """Create new route in TGW route table"""
       cidr_block = event['detail']['responseElements']['cidrBlock']
       attachment_id = event['detail']['responseElements']['attachmentId']
       
       response = ec2_client.create_route(
           RouteTableId=route_table_id,
           DestinationCidrBlock=cidr_block,
           TransitGatewayAttachmentId=attachment_id
       )
       
       return response
   ```

**Step 10: Security and Compliance**
1. **VPC Flow Logs for TGW traffic**:
   - **Resource**: Transit Gateway
   - **Traffic type**: ALL
   - **Destination**: CloudWatch Logs or S3
   - **Log format**: Include TGW-specific fields

2. **Security monitoring**:
   ```python
   # CloudWatch Logs Insights query for TGW security
   fields @timestamp, srcaddr, dstaddr, srcport, dstport, protocol, action
   | filter srcaddr like /^10\./ and dstaddr like /^10\./
   | filter action = "REJECT"
   | stats count() by srcaddr, dstaddr
   | sort count desc
   ```

**Step 11: Cost Optimization**
1. **TGW pricing components**:
   - **Hourly charge**: Per TGW attachment
   - **Data processing**: Per GB processed
   - **Cross-AZ charges**: For TGW traffic

2. **Optimization strategies**:
   - **Consolidate attachments**: Where possible
   - **Optimize routing**: Minimize cross-AZ traffic
   - **Use VPC endpoints**: For AWS services

**Step 12: Backup and Disaster Recovery**
1. **Infrastructure as Code**:
   ```yaml
   TransitGateway:
     Type: AWS::EC2::TransitGateway
     Properties:
       AmazonSideAsn: 64512
       Description: Central transit gateway
       Tags:
         - Key: Name
           Value: central-tgw
   
   ProductionAttachment:
     Type: AWS::EC2::TransitGatewayAttachment
     Properties:
       TransitGatewayId: !Ref TransitGateway
       VpcId: !Ref ProductionVPC
       SubnetIds:
         - !Ref PrivateSubnet1
         - !Ref PrivateSubnet2
   ```

2. **Configuration backup**:
   - **Export route tables**: Regular configuration exports
   - **Document dependencies**: VPC relationships
   - **Test recovery procedures**: Regular DR drills

**Verification Checklist:**
- ✅ Transit Gateway created with proper configuration
- ✅ VPC attachments established and working
- ✅ Custom route tables configured for security
- ✅ Network segmentation implemented
- ✅ VPN connectivity (if applicable) working
- ✅ Cross-region peering (if applicable) configured
- ✅ CloudWatch monitoring and dashboards set up
- ✅ Security monitoring with VPC Flow Logs
- ✅ Cost optimization strategies implemented
- ✅ Infrastructure as Code templates created

### Task 40: AWS Organizations
**Create an AWS Organization with multiple accounts, OUs, and Service Control Policies.**

**Console-Based Solution:**

**Step 1: Create AWS Organization**
1. **Navigate to AWS Organizations Console**
2. **Create organization**:
   - **Feature set**: All features (recommended)
   - **Management account**: Current account becomes management account
3. **Confirm creation**: Accept terms and create organization

**Step 2: Plan Organizational Structure**
1. **Account strategy**:
   - **Management account**: Billing and central governance
   - **Security account**: Central security services and logging
   - **Production accounts**: Per business unit or application
   - **Development accounts**: Per team or environment
   - **Sandbox accounts**: For experimentation

2. **OU (Organizational Unit) structure**:
   ```
   Root OU
   ├── Security OU
   │   ├── Audit Account
   │   └── Log Archive Account
   ├── Production OU
   │   ├── Prod-WebApp Account
   │   └── Prod-Database Account
   ├── Development OU
   │   ├── Dev-Team1 Account
   │   └── Dev-Team2 Account
   └── Sandbox OU
       ├── Sandbox1 Account
       └── Sandbox2 Account
   ```

**Step 3: Create Organizational Units**
1. **Security OU**:
   - **Name**: "Security"
   - **Description**: "Security and compliance accounts"

2. **Production OU**:
   - **Name**: "Production"
   - **Description**: "Production workload accounts"

3. **Development OU**:
   - **Name**: "Development"
   - **Description**: "Development and testing accounts"

4. **Sandbox OU**:
   - **Name**: "Sandbox"
   - **Description**: "Experimental and learning accounts"

**Step 4: Create Member Accounts**
1. **Create Security Account**:
   - **Account name**: "Security-Audit"
   - **Email**: security-audit@yourcompany.com
   - **IAM role name**: OrganizationAccountAccessRole
   - **Move to**: Security OU

2. **Create Production Account**:
   - **Account name**: "Prod-WebApp"
   - **Email**: prod-webapp@yourcompany.com
   - **Move to**: Production OU

3. **Create Development Account**:
   - **Account name**: "Dev-Team1"
   - **Email**: dev-team1@yourcompany.com
   - **Move to**: Development OU

**Step 5: Configure Service Control Policies (SCPs)**
1. **Create Production SCP**:
   - **Policy name**: "ProductionRestrictions"
   - **Description**: "Restrictions for production accounts"
   - **Policy document**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Deny",
         "Action": [
           "ec2:TerminateInstances"
         ],
         "Resource": "*",
         "Condition": {
           "StringNotEquals": {
             "ec2:ResourceTag/Environment": "production"
           }
         }
       },
       {
         "Effect": "Deny",
         "Action": [
           "rds:DeleteDBInstance",
           "rds:DeleteDBCluster"
         ],
         "Resource": "*",
         "Condition": {
           "Bool": {
             "rds:db-instance-deletion-protection": "false"
           }
         }
       },
       {
         "Effect": "Deny",
         "Action": [
           "s3:DeleteBucket",
           "s3:DeleteObject"
         ],
         "Resource": "*",
         "Condition": {
           "StringEquals": {
             "s3:x-amz-mfa": "false"
           }
         }
       }
     ]
   }
   ```

2. **Create Development SCP**:
   - **Policy name**: "DevelopmentRestrictions"
   - **Policy document**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Deny",
         "Action": [
           "ec2:RunInstances"
         ],
         "Resource": "arn:aws:ec2:*:*:instance/*",
         "Condition": {
           "ForAllValues:StringNotEquals": {
             "ec2:InstanceType": [
               "t3.micro",
               "t3.small",
               "t3.medium"
             ]
           }
         }
       },
       {
         "Effect": "Deny",
         "Action": [
           "rds:CreateDBInstance"
         ],
         "Resource": "*",
         "Condition": {
           "ForAllValues:StringNotEquals": {
             "rds:db-instance-class": [
               "db.t3.micro",
               "db.t3.small"
             ]
           }
         }
       }
     ]
   }
   ```

3. **Create Sandbox SCP**:
   - **Policy name**: "SandboxRestrictions"
   - **Policy document**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Deny",
         "Action": [
           "iam:CreateRole",
           "iam:DeleteRole",
           "iam:AttachRolePolicy",
           "iam:DetachRolePolicy"
         ],
         "Resource": "*",
         "Condition": {
           "StringLike": {
             "iam:PolicyArn": "arn:aws:iam::aws:policy/*"
           }
         }
       },
       {
         "Effect": "Deny",
         "Action": [
           "organizations:*",
           "account:*"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

**Step 6: Attach SCPs to OUs**
1. **Production OU**: Attach "ProductionRestrictions" SCP
2. **Development OU**: Attach "DevelopmentRestrictions" SCP
3. **Sandbox OU**: Attach "SandboxRestrictions" SCP

**Step 7: Configure Centralized Logging**
1. **Set up CloudTrail organization trail**:
   - **Trail name**: "organization-cloudtrail"
   - **Apply to organization**: Yes
   - **S3 bucket**: Create in Security account
   - **Log file encryption**: Enable with KMS
   - **Log file validation**: Enable

2. **Configure Config organization aggregator**:
   - **Aggregator name**: "organization-config-aggregator"
   - **Source accounts**: All organization accounts
   - **Source regions**: All regions
   - **Data replication**: Enable

**Step 8: Implement Cross-Account Roles**
1. **Create centralized access roles**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::MANAGEMENT-ACCOUNT:root"
         },
         "Action": "sts:AssumeRole",
         "Condition": {
           "StringEquals": {
             "sts:ExternalId": "unique-external-id"
           },
           "IpAddress": {
             "aws:SourceIp": ["203.0.113.0/24"]
           }
         }
       }
     ]
   }
   ```

2. **Create role assumption script**:
   ```python
   import boto3
   import json
   
   def assume_cross_account_role(account_id, role_name, external_id):
       """
       Assume role in member account
       """
       sts = boto3.client('sts')
       
       role_arn = f"arn:aws:iam::{account_id}:role/{role_name}"
       
       response = sts.assume_role(
           RoleArn=role_arn,
           RoleSessionName='OrganizationAdminSession',
           ExternalId=external_id
       )
       
       credentials = response['Credentials']
       
       # Create session with assumed role credentials
       session = boto3.Session(
           aws_access_key_id=credentials['AccessKeyId'],
           aws_secret_access_key=credentials['SecretAccessKey'],
           aws_session_token=credentials['SessionToken']
       )
       
       return session
   ```

**Step 9: Set Up Consolidated Billing Analysis**
1. **Create Cost and Usage Reports**:
   - **Report name**: "organization-cur"
   - **Time granularity**: Daily
   - **Resource IDs**: Include
   - **Data refresh settings**: Automatically refresh
   - **Report versioning**: Overwrite report

2. **Set up billing alerts**:
   - **Account-level budgets**: Per member account
   - **Service-level budgets**: Per AWS service
   - **OU-level budgets**: Per organizational unit

3. **Create cost allocation tags**:
   ```python
   import boto3
   
   def setup_cost_allocation_tags():
       """
       Configure cost allocation tags across organization
       """
       organizations = boto3.client('organizations')
       
       # Get all accounts
       accounts = organizations.list_accounts()
       
       for account in accounts['Accounts']:
           if account['Status'] == 'ACTIVE':
               setup_account_tags(account['Id'])
   
   def setup_account_tags(account_id):
       """
       Set up standardized tags for an account
       """
       # Assume role in member account
       session = assume_cross_account_role(
           account_id, 
           'OrganizationAccountAccessRole',
           'unique-external-id'
       )
       
       ec2 = session.client('ec2')
       
       # Create tags for cost allocation
       tags = [
           {'Key': 'CostCenter', 'Value': 'Engineering'},
           {'Key': 'Environment', 'Value': 'Production'},
           {'Key': 'Owner', 'Value': 'DevOps-Team'}
       ]
       
       # Apply tags to all resources
       apply_tags_to_resources(ec2, tags)
   ```

**Step 10: Implement Preventive Controls**
1. **Create guardrails SCP**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Deny",
         "Action": [
           "ec2:RunInstances"
         ],
         "Resource": "*",
         "Condition": {
           "StringNotEquals": {
             "aws:RequestedRegion": [
               "us-east-1",
               "us-west-2"
             ]
           }
         }
       },
       {
         "Effect": "Deny",
         "Action": [
           "iam:CreateAccessKey",
           "iam:UpdateAccessKey"
         ],
         "Resource": "*",
         "Condition": {
           "NumericGreaterThan": {
             "aws:TokenIssueTime": "1577836800"
           }
         }
       }
     ]
   }
   ```

**Step 11: Set Up Account Lifecycle Management**
1. **Account creation automation**:
   ```python
   import boto3
   import json
   
   def lambda_handler(event, context):
       """
       Automated account creation and setup
       """
       organizations = boto3.client('organizations')
       
       account_request = event['account_request']
       
       # Create new account
       response = organizations.create_account(
           Email=account_request['email'],
           AccountName=account_request['name'],
           RoleName='OrganizationAccountAccessRole'
       )
       
       # Wait for account creation
       request_id = response['CreateAccountStatus']['Id']
       
       # Monitor creation status
       while True:
           status = organizations.describe_create_account_status(
               CreateAccountRequestId=request_id
           )
           
           if status['CreateAccountStatus']['State'] == 'SUCCEEDED':
               account_id = status['CreateAccountStatus']['AccountId']
               
               # Move to appropriate OU
               move_account_to_ou(organizations, account_id, account_request['ou'])
               
               # Apply SCPs
               apply_account_policies(account_id, account_request['policies'])
               
               break
           elif status['CreateAccountStatus']['State'] == 'FAILED':
               raise Exception("Account creation failed")
   
   def move_account_to_ou(org_client, account_id, ou_name):
       """Move account to specified OU"""
       # Get OU ID by name
       ou_id = get_ou_id_by_name(org_client, ou_name)
       
       # Move account
       org_client.move_account(
           AccountId=account_id,
           SourceParentId='r-xxxx',  # Root OU ID
           DestinationParentId=ou_id
       )
   ```

**Step 12: Monitor Organization Health**
1. **CloudWatch dashboard for organization metrics**:
   - **Account count**: Total active accounts
   - **SCP violations**: Policy violations
   - **Cost trends**: Spending by OU
   - **Compliance status**: Config rule compliance

2. **Automated compliance checking**:
   ```python
   def check_organization_compliance():
       """
       Check organization-wide compliance
       """
       organizations = boto3.client('organizations')
       config = boto3.client('config')
       
       accounts = organizations.list_accounts()
       
       compliance_report = {
           'compliant_accounts': [],
           'non_compliant_accounts': [],
           'violations': []
       }
       
       for account in accounts['Accounts']:
           if account['Status'] == 'ACTIVE':
               compliance_status = check_account_compliance(
                   account['Id']
               )
               
               if compliance_status['compliant']:
                   compliance_report['compliant_accounts'].append(
                       account['Id']
                   )
               else:
                   compliance_report['non_compliant_accounts'].append({
                       'account_id': account['Id'],
                       'violations': compliance_status['violations']
                   })
       
       return compliance_report
   ```

**Verification Checklist:**
- ✅ AWS Organization created with proper feature set
- ✅ Organizational Units structure implemented
- ✅ Member accounts created and organized
- ✅ Service Control Policies created and attached
- ✅ Centralized logging and monitoring configured
- ✅ Cross-account access roles established
- ✅ Consolidated billing and cost management set up
- ✅ Preventive controls and guardrails implemented
- ✅ Account lifecycle management automated
- ✅ Organization health monitoring operational

### Task 41: Cost and Usage Reports
**Set up detailed billing reports and analyze costs using Athena and QuickSight.**

**Console-Based Solution:**

**Step 1: Create Cost and Usage Report**
1. **Navigate to Billing Console** → "Cost & Usage Reports"
2. **Create report**:
   - **Report name**: "detailed-cost-usage-report"
   - **Additional report details**: Include resource IDs
   - **Time granularity**: Daily
   - **Report versioning**: Create new report version
   - **Data refresh settings**: Automatically refresh

**Step 2: Configure S3 Bucket for Reports**
1. **S3 bucket configuration**:
   - **Bucket name**: "company-cost-usage-reports"
   - **Region**: us-east-1 (recommended for cost reports)
   - **Report path prefix**: "cost-reports/"
   - **Compression**: GZIP

2. **Set up bucket policy**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "billingreports.amazonaws.com"
         },
         "Action": [
           "s3:GetBucketAcl",
           "s3:GetBucketPolicy"
         ],
         "Resource": "arn:aws:s3:::company-cost-usage-reports"
       },
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "billingreports.amazonaws.com"
         },
         "Action": "s3:PutObject",
         "Resource": "arn:aws:s3:::company-cost-usage-reports/*"
       }
     ]
   }
   ```

**Step 3: Set Up Athena Database**
1. **Create Athena database**:
   ```sql
   CREATE DATABASE IF NOT EXISTS cost_usage_reports;
   ```

2. **Create table for CUR data**:
   ```sql
   CREATE EXTERNAL TABLE IF NOT EXISTS cost_usage_reports.detailed_cost_usage_report (
     identity_line_item_id string,
     identity_time_interval string,
     bill_invoice_id string,
     bill_billing_entity string,
     bill_bill_type string,
     bill_payer_account_id string,
     bill_billing_period_start_date string,
     bill_billing_period_end_date string,
     line_item_usage_account_id string,
     line_item_line_item_type string,
     line_item_usage_type string,
     line_item_operation string,
     line_item_availability_zone string,
     line_item_resource_id string,
     line_item_usage_start_date string,
     line_item_usage_end_date string,
     line_item_product_code string,
     line_item_usage_amount double,
     line_item_normalization_factor double,
     line_item_normalized_usage_amount double,
     line_item_currency_code string,
     line_item_unblended_rate string,
     line_item_unblended_cost double,
     line_item_blended_rate string,
     line_item_blended_cost double,
     line_item_line_item_description string,
     line_item_tax_type string,
     product_product_name string,
     product_availability_zone string,
     product_current_generation string,
     product_instance_type string,
     product_location string,
     product_operation string,
     product_operating_system string,
     product_physical_processor string,
     product_pre_installed_sw string,
     product_processor_architecture string,
     product_processor_features string,
     product_product_family string,
     product_region string,
     product_servicecode string,
     product_servicename string,
     product_sku string,
     product_storage string,
     product_tenancy string,
     product_usagetype string,
     product_vcpu string,
     product_volume_type string,
     pricing_rate_code string,
     pricing_rate_id string,
     pricing_currency string,
     pricing_public_on_demand_cost double,
     pricing_public_on_demand_rate string,
     pricing_term string,
     pricing_unit string,
     reservation_amazon_resource_name string,
     reservation_availability_zone string,
     reservation_end_time string,
     reservation_modification_status string,
     reservation_normalized_units_per_reservation string,
     reservation_number_of_reservations string,
     reservation_recurring_fee_for_usage double,
     reservation_start_time string,
     reservation_subscription_id string,
     reservation_total_reserved_normalized_units string,
     reservation_total_reserved_units string,
     reservation_units_per_reservation string,
     reservation_unused_amortized_upfront_fee_for_billing_period double,
     reservation_unused_normalized_unit_quantity double,
     reservation_unused_quantity double,
     reservation_unused_recurring_fee double,
     reservation_upfront_fee double,
     savings_plans_total_commitment_to_date double,
     savings_plans_type string,
     savings_plans_payment_option string,
     savings_plans_used_commitment double,
     savings_plans_savings_plan_arn string,
     savings_plans_savings_plan_rate double,
     savings_plans_instance_type_family string,
     savings_plans_region string,
     savings_plans_start_time string,
     savings_plans_end_time string
   )
   PARTITIONED BY (
     year string,
     month string
   )
   STORED AS PARQUET
   LOCATION 's3://company-cost-usage-reports/cost-reports/detailed-cost-usage-report/'
   TBLPROPERTIES (
     'projection.enabled'='true',
     'projection.year.type'='integer',
     'projection.year.range'='2020,2030',
     'projection.year.interval'='1',
     'projection.month.type'='integer',
     'projection.month.range'='1,12',
     'projection.month.interval'='1',
     'projection.month.digits'='2',
     'storage.location.template'='s3://company-cost-usage-reports/cost-reports/detailed-cost-usage-report/${year}/${month}/'
   );
   ```

**Step 4: Create Useful Athena Queries**
1. **Top 10 most expensive services**:
   ```sql
   SELECT 
     product_servicename,
     SUM(line_item_unblended_cost) as total_cost
   FROM cost_usage_reports.detailed_cost_usage_report
   WHERE year = '2025' 
     AND month = '01'
     AND line_item_line_item_type = 'Usage'
   GROUP BY product_servicename
   ORDER BY total_cost DESC
   LIMIT 10;
   ```

2. **Monthly cost trends by service**:
   ```sql
   SELECT 
     product_servicename,
     year,
     month,
     SUM(line_item_unblended_cost) as monthly_cost
   FROM cost_usage_reports.detailed_cost_usage_report
   WHERE year >= '2024'
     AND line_item_line_item_type = 'Usage'
   GROUP BY product_servicename, year, month
   ORDER BY year, month, monthly_cost DESC;
   ```

3. **Resource-level cost analysis**:
   ```sql
   SELECT 
     line_item_resource_id,
     product_servicename,
     product_instance_type,
     line_item_availability_zone,
     SUM(line_item_unblended_cost) as resource_cost,
     SUM(line_item_usage_amount) as usage_amount,
     line_item_usage_type
   FROM cost_usage_reports.detailed_cost_usage_report
   WHERE year = '2025' 
     AND month = '01'
     AND line_item_resource_id IS NOT NULL
     AND line_item_line_item_type = 'Usage'
   GROUP BY 
     line_item_resource_id,
     product_servicename,
     product_instance_type,
     line_item_availability_zone,
     line_item_usage_type
   ORDER BY resource_cost DESC
   LIMIT 50;
   ```

4. **Reserved Instance utilization**:
   ```sql
   SELECT 
     reservation_subscription_id,
     product_instance_type,
     reservation_availability_zone,
     SUM(reservation_unused_quantity) as unused_hours,
     SUM(reservation_total_reserved_units) as total_reserved_hours,
     (SUM(reservation_unused_quantity) / SUM(reservation_total_reserved_units)) * 100 as unused_percentage
   FROM cost_usage_reports.detailed_cost_usage_report
   WHERE year = '2025' 
     AND month = '01'
     AND line_item_line_item_type = 'RIFee'
   GROUP BY 
     reservation_subscription_id,
     product_instance_type,
     reservation_availability_zone
   ORDER BY unused_percentage DESC;
   ```

**Step 5: Set Up QuickSight for Visualization**
1. **Enable QuickSight**:
   - **Edition**: Enterprise (for advanced features)
   - **Authentication**: IAM only
   - **Region**: us-east-1
   - **IAM role**: Create new QuickSight service role

2. **Grant S3 access to QuickSight**:
   - **Permissions**: Select S3 buckets
   - **Bucket**: company-cost-usage-reports
   - **Read access**: Enable

3. **Create Athena data source**:
   - **Data source name**: "Cost-Usage-Reports"
   - **Database**: cost_usage_reports
   - **Table**: detailed_cost_usage_report
   - **Import to SPICE**: No (query directly)

**Step 6: Create QuickSight Dashboards**
1. **Executive Cost Dashboard**:
   - **Total monthly costs**: Line chart over time
   - **Cost by service**: Pie chart
   - **Cost by account**: Bar chart (if using Organizations)
   - **Budget vs actual**: KPI visual

2. **Operational Cost Dashboard**:
   - **Cost by resource**: Tree map
   - **Instance utilization**: Heat map
   - **Reserved Instance coverage**: Gauge chart
   - **Cost anomalies**: Scatter plot

3. **Department Cost Dashboard**:
   - **Cost by tag**: Filtered by department
   - **Monthly trends**: Department-specific
   - **Resource breakdown**: Detailed drill-down

**Step 7: Automate Cost Analysis**
1. **Lambda function for cost alerts**:
   ```python
   import boto3
   import json
   from datetime import datetime, timedelta
   
   def lambda_handler(event, context):
       """
       Automated cost analysis and alerting
       """
       athena = boto3.client('athena')
       sns = boto3.client('sns')
       
       # Query for cost anomalies
       anomalies = detect_cost_anomalies(athena)
       
       if anomalies:
           send_cost_alert(sns, anomalies)
       
       # Generate weekly cost report
       if datetime.now().weekday() == 0:  # Monday
           weekly_report = generate_weekly_report(athena)
           send_weekly_report(sns, weekly_report)
   
   def detect_cost_anomalies(athena_client):
       """
       Detect unusual cost patterns
       """
       query = """
       WITH daily_costs AS (
         SELECT 
           line_item_usage_start_date,
           SUM(line_item_unblended_cost) as daily_cost
         FROM cost_usage_reports.detailed_cost_usage_report
         WHERE year = '2025' 
           AND month = '01'
           AND line_item_line_item_type = 'Usage'
         GROUP BY line_item_usage_start_date
       ),
       cost_stats AS (
         SELECT 
           AVG(daily_cost) as avg_cost,
           STDDEV(daily_cost) as stddev_cost
         FROM daily_costs
       )
       SELECT 
         dc.line_item_usage_start_date,
         dc.daily_cost,
         cs.avg_cost,
         cs.stddev_cost,
         CASE 
           WHEN dc.daily_cost > (cs.avg_cost + 2 * cs.stddev_cost) THEN 'HIGH_ANOMALY'
           WHEN dc.daily_cost > (cs.avg_cost + cs.stddev_cost) THEN 'MEDIUM_ANOMALY'
           ELSE 'NORMAL'
         END as anomaly_level
       FROM daily_costs dc
       CROSS JOIN cost_stats cs
       WHERE dc.daily_cost > (cs.avg_cost + cs.stddev_cost)
       ORDER BY dc.daily_cost DESC;
       """
       
       # Execute query and return results
       return execute_athena_query(athena_client, query)
   
   def execute_athena_query(athena_client, query):
       """
       Execute Athena query and return results
       """
       response = athena_client.start_query_execution(
           QueryString=query,
           QueryExecutionContext={
               'Database': 'cost_usage_reports'
           },
           ResultConfiguration={
               'OutputLocation': 's3://athena-query-results-bucket/'
           }
       )
       
       # Wait for query completion and return results
       # Implementation details omitted for brevity
       pass
   ```

**Step 8: Set Up Cost Optimization Recommendations**
1. **Create optimization queries**:
   ```sql
   -- Underutilized EC2 instances
   SELECT 
     line_item_resource_id,
     product_instance_type,
     AVG(line_item_usage_amount) as avg_hours_per_day,
     SUM(line_item_unblended_cost) as monthly_cost
   FROM cost_usage_reports.detailed_cost_usage_report
   WHERE year = '2025' 
     AND month = '01'
     AND product_servicename = 'Amazon Elastic Compute Cloud - Compute'
     AND line_item_usage_type LIKE '%BoxUsage%'
   GROUP BY line_item_resource_id, product_instance_type
   HAVING AVG(line_item_usage_amount) < 20  -- Less than 20 hours per day
   ORDER BY monthly_cost DESC;
   ```

2. **Reserved Instance recommendations**:
   ```sql
   -- Identify candidates for Reserved Instances
   SELECT 
     product_instance_type,
     product_region,
     COUNT(DISTINCT line_item_resource_id) as instance_count,
     SUM(line_item_unblended_cost) as total_cost,
     SUM(line_item_unblended_cost) * 0.7 as estimated_ri_cost
   FROM cost_usage_reports.detailed_cost_usage_report
   WHERE year = '2025' 
     AND line_item_usage_type LIKE '%BoxUsage%'
     AND line_item_line_item_type = 'Usage'
   GROUP BY product_instance_type, product_region
   HAVING COUNT(DISTINCT line_item_resource_id) >= 3  -- At least 3 instances
     AND SUM(line_item_unblended_cost) > 100  -- Monthly cost > $100
   ORDER BY total_cost DESC;
   ```

**Step 9: Create Cost Governance Framework**
1. **Tag-based cost allocation**:
   ```sql
   SELECT 
     resource_tags_user_cost_center as cost_center,
     resource_tags_user_environment as environment,
     product_servicename,
     SUM(line_item_unblended_cost) as allocated_cost
   FROM cost_usage_reports.detailed_cost_usage_report
   WHERE year = '2025' 
     AND month = '01'
     AND resource_tags_user_cost_center IS NOT NULL
   GROUP BY 
     resource_tags_user_cost_center,
     resource_tags_user_environment,
     product_servicename
   ORDER BY allocated_cost DESC;
   ```

2. **Cost center reporting automation**:
   ```python
   def generate_cost_center_reports():
       """
       Generate automated cost reports for each cost center
       """
       athena = boto3.client('athena')
       s3 = boto3.client('s3')
       
       # Get list of cost centers
       cost_centers = get_cost_centers(athena)
       
       for cost_center in cost_centers:
           # Generate report for each cost center
           report_data = generate_cost_center_report(athena, cost_center)
           
           # Save to S3 for distribution
           save_report_to_s3(s3, cost_center, report_data)
           
           # Send via email
           send_cost_center_report(cost_center, report_data)
   ```

**Step 10: Monitor and Alert on Cost Changes**
1. **CloudWatch custom metrics**:
   ```python
   def publish_cost_metrics():
       """
       Publish custom cost metrics to CloudWatch
       """
       cloudwatch = boto3.client('cloudwatch')
       athena = boto3.client('athena')
       
       # Get daily cost
       daily_cost = get_daily_cost(athena)
       
       # Publish to CloudWatch
       cloudwatch.put_metric_data(
           Namespace='CostManagement',
           MetricData=[
               {
                   'MetricName': 'DailyCost',
                   'Value': daily_cost,
                   'Unit': 'None',
                   'Dimensions': [
                       {
                           'Name': 'Account',
                           'Value': boto3.Session().region_name
                       }
                   ]
               }
           ]
       )
   ```

2. **Cost threshold alarms**:
   - **Daily cost > baseline**: Alert on unusual daily spending
   - **Monthly projection**: Alert if trending over budget
   - **Service cost spike**: Alert on specific service increases

**Verification Checklist:**
- ✅ Cost and Usage Reports configured with daily granularity
- ✅ S3 bucket set up with proper permissions
- ✅ Athena database and tables created
- ✅ Useful cost analysis queries developed
- ✅ QuickSight dashboards created for different audiences
- ✅ Automated cost analysis and alerting implemented
- ✅ Cost optimization recommendations generated
- ✅ Tag-based cost allocation working
- ✅ Cost governance framework established
- ✅ CloudWatch metrics and alarms configured

### Task 42: Lambda Layers
**Create and use Lambda layers for code reusability and dependency management.**

**Console-Based Solution:**

**Step 1: Navigate to Lambda Layers Console**
1. Go to **AWS Lambda Console**
2. Click "Layers" in left navigation
3. Click "Create layer"

**Step 2: Prepare Layer Content**
1. **Create local directory structure**:
   ```bash
   mkdir lambda-layer
   cd lambda-layer
   mkdir python/lib/python3.9/site-packages
   ```

2. **Install dependencies**:
   ```bash
   pip install requests boto3 pandas -t python/lib/python3.9/site-packages/
   ```

3. **Create custom utility modules**:
   ```python
   # python/lib/python3.9/site-packages/utils/helpers.py
   import json
   import logging
   from datetime import datetime
   
   def setup_logging(level='INFO'):
       """Setup standardized logging"""
       logging.basicConfig(
           level=getattr(logging, level),
           format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
       )
       return logging.getLogger(__name__)
   
   def response_formatter(status_code, body, headers=None):
       """Format Lambda response"""
       default_headers = {
           'Content-Type': 'application/json',
           'Access-Control-Allow-Origin': '*'
       }
       if headers:
           default_headers.update(headers)
       
       return {
           'statusCode': status_code,
           'headers': default_headers,
           'body': json.dumps(body) if isinstance(body, dict) else body
       }
   
   def validate_required_fields(data, required_fields):
       """Validate required fields in request data"""
       missing_fields = [field for field in required_fields if field not in data]
       if missing_fields:
           raise ValueError(f"Missing required fields: {missing_fields}")
       return True
   ```

4. **Create configuration utilities**:
   ```python
   # python/lib/python3.9/site-packages/config/settings.py
   import os
   import boto3
   
   class Config:
       def __init__(self):
           self.environment = os.getenv('ENVIRONMENT', 'development')
           self.aws_region = os.getenv('AWS_REGION', 'us-east-1')
           self.log_level = os.getenv('LOG_LEVEL', 'INFO')
       
       def get_parameter(self, parameter_name):
           """Get parameter from Systems Manager Parameter Store"""
           ssm = boto3.client('ssm', region_name=self.aws_region)
           response = ssm.get_parameter(
               Name=parameter_name,
               WithDecryption=True
           )
           return response['Parameter']['Value']
       
       def get_secret(self, secret_name):
           """Get secret from Secrets Manager"""
           secrets = boto3.client('secretsmanager', region_name=self.aws_region)
           response = secrets.get_secret_value(SecretId=secret_name)
           return json.loads(response['SecretString'])
   ```

**Step 3: Package Layer**
1. **Create deployment package**:
   ```bash
   zip -r lambda-layer.zip python/
   ```

2. **Verify package structure**:
   ```bash
   unzip -l lambda-layer.zip
   # Should show python/lib/python3.9/site-packages/... structure
   ```

**Step 4: Create Layer in AWS**
1. **Layer configuration**:
   - **Name**: "common-utilities-layer"
   - **Description**: "Common utilities and dependencies for Lambda functions"
   - **Upload**: Choose file → Select lambda-layer.zip
   - **Compatible runtimes**: python3.9, python3.10, python3.11
   - **Compatible architectures**: x86_64, arm64
   - **License**: MIT (optional)

2. **Layer permissions**:
   - **Add permissions**: Enable
   - **Account IDs**: Your account ID or organization accounts
   - **Organization ID**: If using AWS Organizations

**Step 5: Create Lambda Function Using Layer**
1. **Create new Lambda function**:
   - **Function name**: "layer-test-function"
   - **Runtime**: Python 3.9
   - **Function code**:
   ```python
   import json
   from utils.helpers import setup_logging, response_formatter, validate_required_fields
   from config.settings import Config
   import requests
   import pandas as pd
   
   # Initialize
   logger = setup_logging()
   config = Config()
   
   def lambda_handler(event, context):
       try:
           logger.info(f"Processing event: {event}")
           
           # Validate input
           validate_required_fields(event, ['action'])
           
           action = event['action']
           
           if action == 'fetch_data':
               # Use requests library from layer
               response = requests.get('https://api.github.com/users/octocat')
               data = response.json()
               
               return response_formatter(200, {
                   'message': 'Data fetched successfully',
                   'data': data
               })
           
           elif action == 'process_data':
               # Use pandas from layer
               df = pd.DataFrame(event.get('data', []))
               result = {
                   'row_count': len(df),
                   'columns': list(df.columns) if not df.empty else []
               }
               
               return response_formatter(200, {
                   'message': 'Data processed successfully',
                   'result': result
               })
           
           else:
               return response_formatter(400, {
                   'error': f'Unknown action: {action}'
               })
               
       except ValueError as e:
           logger.error(f"Validation error: {e}")
           return response_formatter(400, {'error': str(e)})
       
       except Exception as e:
           logger.error(f"Unexpected error: {e}")
           return response_formatter(500, {'error': 'Internal server error'})
   ```

**Step 6: Attach Layer to Function**
1. **Function configuration** → "Layers"
2. **Add layer**:
   - **Layer source**: Custom layers
   - **Custom layers**: common-utilities-layer
   - **Version**: 1 (latest)

**Step 7: Test Function with Layer**
1. **Create test event**:
   ```json
   {
     "action": "fetch_data"
   }
   ```

2. **Execute test** and verify layer dependencies work

**Step 8: Create Version-Specific Layers**
1. **Development layer**:
   - **Name**: "dev-utilities-layer"
   - **Include**: Debug tools, development libraries
   
2. **Production layer**:
   - **Name**: "prod-utilities-layer"
   - **Include**: Only production dependencies
   - **Optimized**: Smaller package size

**Step 9: Implement Layer Versioning Strategy**
1. **Version naming convention**:
   - v1.0.0: Major.Minor.Patch
   - Include changelog in description
   
2. **Layer update process**:
   ```bash
   # Update layer code
   vim python/lib/python3.9/site-packages/utils/helpers.py
   
   # Package new version
   zip -r lambda-layer-v1.1.0.zip python/
   
   # Upload as new layer version
   aws lambda publish-layer-version \
     --layer-name common-utilities-layer \
     --description "v1.1.0 - Added new validation functions" \
     --zip-file fileb://lambda-layer-v1.1.0.zip \
     --compatible-runtimes python3.9 python3.10 python3.11
   ```

**Step 10: Manage Layer Permissions**
1. **Cross-account sharing**:
   ```bash
   aws lambda add-layer-version-permission \
     --layer-name common-utilities-layer \
     --version-number 1 \
     --statement-id cross-account-access \
     --principal 123456789012 \
     --action lambda:GetLayerVersion
   ```

2. **Organization-wide sharing**:
   ```bash
   aws lambda add-layer-version-permission \
     --layer-name common-utilities-layer \
     --version-number 1 \
     --statement-id org-access \
     --principal "*" \
     --action lambda:GetLayerVersion \
     --organization-id o-1234567890
   ```

**Step 11: Monitor Layer Usage**
1. **CloudWatch metrics**:
   - Layer download frequency
   - Functions using specific layer versions
   - Performance impact measurement

2. **Usage tracking script**:
   ```python
   import boto3
   
   def audit_layer_usage(layer_name):
       lambda_client = boto3.client('lambda')
       
       # Get all functions
       functions = lambda_client.list_functions()
       
       layer_usage = []
       for function in functions['Functions']:
           layers = function.get('Layers', [])
           for layer in layers:
               if layer_name in layer['Arn']:
                   layer_usage.append({
                       'function_name': function['FunctionName'],
                       'layer_version': layer['Arn'].split(':')[-1],
                       'last_modified': function['LastModified']
                   })
       
       return layer_usage
   ```

**Step 12: Best Practices Implementation**
1. **Layer optimization**:
   - Keep layers under 50MB (unzipped)
   - Include only necessary dependencies
   - Use multiple layers for different purposes

2. **Security considerations**:
   - Regular dependency updates
   - Vulnerability scanning
   - Access control via IAM

**Step 13: Automated Layer Deployment**
1. **CI/CD pipeline for layers**:
   ```yaml
   # buildspec.yml for CodeBuild
   version: 0.2
   phases:
     install:
       runtime-versions:
         python: 3.9
     pre_build:
       commands:
         - mkdir -p layer/python/lib/python3.9/site-packages
         - pip install -r requirements.txt -t layer/python/lib/python3.9/site-packages/
     build:
       commands:
         - cd layer && zip -r ../layer.zip python/
         - aws lambda publish-layer-version --layer-name $LAYER_NAME --zip-file fileb://layer.zip
   ```

**Verification Checklist:**
- ✅ Lambda layer created with proper structure
- ✅ Dependencies packaged correctly
- ✅ Layer attached to test function successfully
- ✅ Custom utilities accessible in function code
- ✅ Version management strategy implemented
- ✅ Cross-account permissions configured (if needed)
- ✅ Usage monitoring and tracking set up
- ✅ CI/CD pipeline for layer updates created

### Task 43: Step Functions
**Build a serverless workflow using AWS Step Functions with complex business logic.**

**Console-Based Solution:**

**Step 1: Navigate to Step Functions Console**
1. Go to **AWS Step Functions Console**
2. Click "Create state machine"

**Step 2: Plan Workflow Architecture**
1. **Business process example**: Order processing workflow
   - Validate order
   - Process payment
   - Update inventory
   - Send notifications
   - Handle failures

2. **State types to use**:
   - **Task**: Lambda function execution
   - **Choice**: Conditional branching
   - **Parallel**: Concurrent execution
   - **Wait**: Time delays
   - **Fail/Succeed**: Terminal states

**Step 3: Create Supporting Lambda Functions**
1. **Order validation function**:
   ```python
   import json
   
   def lambda_handler(event, context):
       order = event['order']
       
       # Validation logic
       errors = []
       
       if not order.get('customer_id'):
           errors.append('Missing customer_id')
       
       if not order.get('items') or len(order['items']) == 0:
           errors.append('No items in order')
       
       for item in order.get('items', []):
           if not item.get('product_id') or not item.get('quantity'):
               errors.append(f'Invalid item: {item}')
       
       if errors:
           raise Exception(f"Validation failed: {', '.join(errors)}")
       
       return {
           'order': order,
           'validation_status': 'passed',
           'validated_at': context.aws_request_id
       }
   ```

2. **Payment processing function**:
   ```python
   import json
   import random
   
   def lambda_handler(event, context):
       order = event['order']
       
       # Simulate payment processing
       payment_amount = sum(item['price'] * item['quantity'] for item in order['items'])
       
       # Simulate random payment failure (10% chance)
       if random.random() < 0.1:
           return {
               'payment_status': 'failed',
               'payment_id': None,
               'error': 'Payment gateway timeout',
               'order': order
           }
       
       payment_id = f"pay_{context.aws_request_id[:8]}"
       
       return {
           'payment_status': 'success',
           'payment_id': payment_id,
           'amount': payment_amount,
           'order': order
       }
   ```

3. **Inventory update function**:
   ```python
   import json
   import boto3
   
   def lambda_handler(event, context):
       dynamodb = boto3.resource('dynamodb')
       table = dynamodb.Table('inventory')
       
       order = event['order']
       
       try:
           # Update inventory for each item
           inventory_updates = []
           
           for item in order['items']:
               response = table.update_item(
                   Key={'product_id': item['product_id']},
                   UpdateExpression='SET quantity = quantity - :qty',
                   ExpressionAttributeValues={':qty': item['quantity']},
                   ConditionExpression='quantity >= :qty',
                   ReturnValues='UPDATED_NEW'
               )
               inventory_updates.append({
                   'product_id': item['product_id'],
                   'updated_quantity': response['Attributes']['quantity']
               })
           
           return {
               'inventory_status': 'updated',
               'inventory_updates': inventory_updates,
               'order': order
           }
           
       except Exception as e:
           return {
               'inventory_status': 'failed',
               'error': str(e),
               'order': order
           }
   ```

4. **Notification function**:
   ```python
   import json
   import boto3
   
   def lambda_handler(event, context):
       sns = boto3.client('sns')
       
       order = event['order']
       topic_arn = 'arn:aws:sns:us-east-1:123456789012:order-notifications'
       
       message = {
           'order_id': order['order_id'],
           'customer_id': order['customer_id'],
           'status': event.get('status', 'completed'),
           'message': event.get('message', 'Order processed successfully')
       }
       
       sns.publish(
           TopicArn=topic_arn,
           Message=json.dumps(message),
           Subject=f"Order Update: {order['order_id']}"
       )
       
       return {
           'notification_sent': True,
           'order': order
       }
   ```

**Step 4: Create State Machine Definition**
1. **Choose authoring method**: Write your workflow in code
2. **Definition type**: Standard
3. **State machine definition**:
   ```json
   {
     "Comment": "Order Processing Workflow",
     "StartAt": "ValidateOrder",
     "States": {
       "ValidateOrder": {
         "Type": "Task",
         "Resource": "arn:aws:lambda:us-east-1:123456789012:function:validate-order",
         "Retry": [
           {
             "ErrorEquals": ["Lambda.ServiceException", "Lambda.AWSLambdaException"],
             "IntervalSeconds": 2,
             "MaxAttempts": 3,
             "BackoffRate": 2.0
           }
         ],
         "Catch": [
           {
             "ErrorEquals": ["States.ALL"],
             "Next": "OrderValidationFailed",
             "ResultPath": "$.error"
           }
         ],
         "Next": "ProcessPayment"
       },
       
       "ProcessPayment": {
         "Type": "Task",
         "Resource": "arn:aws:lambda:us-east-1:123456789012:function:process-payment",
         "Next": "CheckPaymentStatus"
       },
       
       "CheckPaymentStatus": {
         "Type": "Choice",
         "Choices": [
           {
             "Variable": "$.payment_status",
             "StringEquals": "success",
             "Next": "ProcessInventoryAndNotification"
           },
           {
             "Variable": "$.payment_status",
             "StringEquals": "failed",
             "Next": "PaymentFailed"
           }
         ],
         "Default": "PaymentFailed"
       },
       
       "ProcessInventoryAndNotification": {
         "Type": "Parallel",
         "Branches": [
           {
             "StartAt": "UpdateInventory",
             "States": {
               "UpdateInventory": {
                 "Type": "Task",
                 "Resource": "arn:aws:lambda:us-east-1:123456789012:function:update-inventory",
                 "End": true
               }
             }
           },
           {
             "StartAt": "SendSuccessNotification",
             "States": {
               "SendSuccessNotification": {
                 "Type": "Task",
                 "Resource": "arn:aws:lambda:us-east-1:123456789012:function:send-notification",
                 "Parameters": {
                   "order.$": "$.order",
                   "status": "success",
                   "message": "Order processed successfully"
                 },
                 "End": true
               }
             }
           }
         ],
         "Next": "OrderCompleted"
       },
       
       "OrderCompleted": {
         "Type": "Pass",
         "Result": {
           "status": "completed",
           "message": "Order processing completed successfully"
         },
         "End": true
       },
       
       "OrderValidationFailed": {
         "Type": "Task",
         "Resource": "arn:aws:lambda:us-east-1:123456789012:function:send-notification",
         "Parameters": {
           "order.$": "$.order",
           "status": "validation_failed",
           "message.$": "$.error.Cause"
         },
         "Next": "OrderFailed"
       },
       
       "PaymentFailed": {
         "Type": "Task",
         "Resource": "arn:aws:lambda:us-east-1:123456789012:function:send-notification",
         "Parameters": {
           "order.$": "$.order",
           "status": "payment_failed",
           "message": "Payment processing failed"
         },
         "Next": "OrderFailed"
       },
       
       "OrderFailed": {
         "Type": "Fail",
         "Cause": "Order processing failed"
       }
     }
   }
   ```

**Step 5: Configure State Machine Settings**
1. **State machine settings**:
   - **Name**: "OrderProcessingWorkflow"
   - **Type**: Standard (for audit trail and debugging)
   - **Role**: Create new IAM role or use existing
   - **Logging**: Enable CloudWatch Logs
   - **X-Ray tracing**: Enable for debugging

**Step 6: Create IAM Role for State Machine**
1. **Create execution role**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "lambda:InvokeFunction"
         ],
         "Resource": [
           "arn:aws:lambda:us-east-1:123456789012:function:validate-order",
           "arn:aws:lambda:us-east-1:123456789012:function:process-payment",
           "arn:aws:lambda:us-east-1:123456789012:function:update-inventory",
           "arn:aws:lambda:us-east-1:123456789012:function:send-notification"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "logs:CreateLogDelivery",
           "logs:GetLogDelivery",
           "logs:UpdateLogDelivery",
           "logs:DeleteLogDelivery",
           "logs:ListLogDeliveries",
           "logs:PutResourcePolicy",
           "logs:DescribeResourcePolicies",
           "logs:DescribeLogGroups"
         ],
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "xray:PutTraceSegments",
           "xray:PutTelemetryRecords",
           "xray:GetSamplingRules",
           "xray:GetSamplingTargets"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

**Step 7: Test State Machine**
1. **Create test execution**:
   ```json
   {
     "order": {
       "order_id": "ord_123456",
       "customer_id": "cust_789",
       "items": [
         {
           "product_id": "prod_001",
           "quantity": 2,
           "price": 29.99
         },
         {
           "product_id": "prod_002",
           "quantity": 1,
           "price": 49.99
         }
       ]
     }
   }
   ```

2. **Monitor execution**:
   - **Graph view**: Visual execution flow
   - **Step details**: Input/output for each step
   - **Execution events**: Detailed timeline
   - **CloudWatch logs**: Detailed logging

**Step 8: Implement Error Handling Patterns**
1. **Circuit breaker pattern**:
   ```json
   {
     "Type": "Task",
     "Resource": "arn:aws:lambda:function:external-api-call",
     "Retry": [
       {
         "ErrorEquals": ["States.TaskFailed"],
         "IntervalSeconds": 1,
         "MaxAttempts": 3,
         "BackoffRate": 2.0
       }
     ],
     "Catch": [
       {
         "ErrorEquals": ["States.ALL"],
         "Next": "FallbackProcess"
       }
     ]
   }
   ```

2. **Compensation pattern** for rollbacks:
   ```json
   {
     "Type": "Parallel",
     "Branches": [
       {
         "StartAt": "ProcessOrder",
         "States": {
           "ProcessOrder": { "Type": "Task", "End": true }
         }
       }
     ],
     "Catch": [
       {
         "ErrorEquals": ["States.ALL"],
         "Next": "CompensateOrder"
       }
     ]
   }
   ```

**Step 9: Add Advanced Features**
1. **Dynamic parallelism** with Map state:
   ```json
   {
     "Type": "Map",
     "ItemsPath": "$.order.items",
     "MaxConcurrency": 5,
     "Iterator": {
       "StartAt": "ProcessItem",
       "States": {
         "ProcessItem": {
           "Type": "Task",
           "Resource": "arn:aws:lambda:function:process-item",
           "End": true
         }
       }
     },
     "Next": "CombineResults"
   }
   ```

2. **Wait states for delays**:
   ```json
   {
     "Type": "Wait",
     "Seconds": 300,
     "Next": "CheckStatus"
   }
   ```

**Step 10: Monitor and Optimize**
1. **CloudWatch metrics**:
   - Execution duration
   - Success/failure rates
   - Cost per execution
   - State transition counts

2. **Performance optimization**:
   - Reduce Lambda cold starts
   - Optimize state transitions
   - Use Express workflows for high-volume scenarios

**Step 11: Implement Workflow Versioning**
1. **Create workflow versions**:
   ```bash
   aws stepfunctions create-state-machine \
     --name OrderProcessingWorkflow-v2 \
     --definition file://state-machine-v2.json \
     --role-arn arn:aws:iam::123456789012:role/StepFunctionsRole
   ```

2. **Blue/Green deployments**:
   - Deploy new version alongside existing
   - Gradually shift traffic
   - Monitor and rollback if needed

**Verification Checklist:**
- ✅ State machine created with complex business logic
- ✅ Multiple Lambda functions integrated
- ✅ Error handling and retry mechanisms implemented
- ✅ Parallel processing configured
- ✅ Choice states for conditional logic working
- ✅ CloudWatch logging and X-Ray tracing enabled
- ✅ Test executions completed successfully
- ✅ Performance monitoring set up
- ✅ Versioning strategy implemented

### Task 44: EKS Cluster
**Deploy and manage a production-ready Kubernetes cluster using Amazon EKS.**

**Console-Based Solution:**

**Step 1: Prerequisites Setup**
1. **Install kubectl**:
   ```bash
   curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.24.7/2022-10-31/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   sudo mv ./kubectl /usr/local/bin
   ```

2. **Install AWS CLI** (if not already installed)
3. **Create EKS service role**:
   - Go to **IAM Console** → "Roles" → "Create role"
   - **Service**: EKS - Cluster
   - **Policies**: AmazonEKSClusterPolicy

**Step 2: Navigate to EKS Console**
1. Go to **Amazon EKS Console**
2. Click "Create cluster"

**Step 3: Configure Cluster Basics**
1. **Cluster configuration**:
   - **Name**: "production-eks-cluster"
   - **Kubernetes version**: 1.24 (latest stable)
   - **Cluster service role**: Select EKS service role created above
   - **Tags**:
     - Environment: production
     - Project: webapp
     - Team: platform

**Step 4: Configure Networking**
1. **VPC and subnets**:
   - **VPC**: Select existing VPC with private/public subnets
   - **Subnets**: Select both private and public subnets across multiple AZs
   - **Security groups**: Default (additional SGs can be added later)

2. **Cluster endpoint access**:
   - **Public and private**: Recommended for production
   - **Public access sources**: Restrict to specific CIDR ranges
   - **Private access sources**: VPC CIDR

**Step 5: Configure Control Plane Logging**
1. **Enable logging** for:
   - ✅ API server
   - ✅ Audit
   - ✅ Authenticator
   - ✅ Controller manager
   - ✅ Scheduler

2. **CloudWatch log group**: /aws/eks/production-eks-cluster/cluster

**Step 6: Create Cluster**
1. **Review configuration** and click "Create"
2. **Wait for cluster creation** (10-15 minutes)
3. **Status should show**: Active

**Step 7: Configure kubectl**
1. **Update kubeconfig**:
   ```bash
   aws eks update-kubeconfig --region us-east-1 --name production-eks-cluster
   ```

2. **Verify connection**:
   ```bash
   kubectl get svc
   # Should show kubernetes service
   ```

**Step 8: Create Node Groups**
1. **Navigate to cluster** → "Compute" tab
2. **Add node group**:
   - **Node group name**: "primary-nodes"
   - **Node IAM role**: Create new role with policies:
     - AmazonEKSWorkerNodePolicy
     - AmazonEKS_CNI_Policy
     - AmazonEC2ContainerRegistryReadOnly

**Step 9: Configure Node Group Details**
1. **Node group compute configuration**:
   - **AMI type**: Amazon Linux 2 (AL2_x86_64)
   - **Capacity type**: On-Demand
   - **Instance types**: t3.medium, t3.large
   - **Disk size**: 20 GB
   - **Node scaling configuration**:
     - **Desired size**: 3
     - **Minimum size**: 1
     - **Maximum size**: 10

2. **Node group network configuration**:
   - **Subnets**: Private subnets only
   - **Allow remote access**: Enable
   - **EC2 Key Pair**: Select existing key pair
   - **Source security groups**: Allow from control plane

**Step 10: Install Essential Add-ons**
1. **Install AWS Load Balancer Controller**:
   ```bash
   # Create service account
   kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml
   
   # Install AWS Load Balancer Controller
   kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
   
   helm repo add eks https://aws.github.io/eks-charts
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
     -n kube-system \
     --set clusterName=production-eks-cluster \
     --set serviceAccount.create=false \
     --set serviceAccount.name=aws-load-balancer-controller
   ```

2. **Install Cluster Autoscaler**:
   ```yaml
   # cluster-autoscaler.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: cluster-autoscaler
     namespace: kube-system
   spec:
     selector:
       matchLabels:
         app: cluster-autoscaler
     template:
       metadata:
         labels:
           app: cluster-autoscaler
       spec:
         serviceAccountName: cluster-autoscaler
         containers:
         - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.24.0
           name: cluster-autoscaler
           resources:
             limits:
               cpu: 100m
               memory: 300Mi
             requests:
               cpu: 100m
               memory: 300Mi
           command:
           - ./cluster-autoscaler
           - --v=4
           - --stderrthreshold=info
           - --cloud-provider=aws
           - --skip-nodes-with-local-storage=false
           - --expander=least-waste
           - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/production-eks-cluster
   ```

**Step 11: Configure RBAC and Security**
1. **Create service account for applications**:
   ```yaml
   # service-account.yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: webapp-service-account
     namespace: default
     annotations:
       eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/WebAppRole
   ```

2. **Create RBAC policies**:
   ```yaml
   # rbac.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: default
     name: webapp-role
   rules:
   - apiGroups: [""]
     resources: ["pods", "services", "configmaps"]
     verbs: ["get", "list", "create", "update", "patch", "delete"]
   
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: webapp-rolebinding
     namespace: default
   subjects:
   - kind: ServiceAccount
     name: webapp-service-account
     namespace: default
   roleRef:
     kind: Role
     name: webapp-role
     apiGroup: rbac.authorization.k8s.io
   ```

**Step 12: Deploy Sample Application**
1. **Create deployment**:
   ```yaml
   # webapp-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: webapp-deployment
     labels:
       app: webapp
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: webapp
     template:
       metadata:
         labels:
           app: webapp
       spec:
         serviceAccountName: webapp-service-account
         containers:
         - name: webapp
           image: nginx:1.21
           ports:
           - containerPort: 80
           resources:
             requests:
               memory: "64Mi"
               cpu: "100m"
             limits:
               memory: "128Mi"
               cpu: "200m"
           readinessProbe:
             httpGet:
               path: /
               port: 80
             initialDelaySeconds: 5
             periodSeconds: 10
           livenessProbe:
             httpGet:
               path: /
               port: 80
             initialDelaySeconds: 30
             periodSeconds: 30
   ```

2. **Create service**:
   ```yaml
   # webapp-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: webapp-service
     annotations:
       service.beta.kubernetes.io/aws-load-balancer-type: nlb
   spec:
     type: LoadBalancer
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
     selector:
       app: webapp
   ```

**Step 13: Configure Monitoring**
1. **Install Prometheus and Grafana**:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   
   helm install prometheus prometheus-community/kube-prometheus-stack \
     --namespace monitoring \
     --create-namespace \
     --set prometheus.prometheusSpec.retention=30d \
     --set grafana.adminPassword=admin123
   ```

2. **Configure CloudWatch Container Insights**:
   ```bash
   curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/production-eks-cluster/;s/{{region_name}}/us-east-1/" | kubectl apply -f -
   ```

**Step 14: Implement GitOps with ArgoCD**
1. **Install ArgoCD**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   
   # Expose ArgoCD server
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
   ```

2. **Configure GitOps repository**:
   ```yaml
   # application.yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: webapp-production
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: https://github.com/company/k8s-manifests
       targetRevision: main
       path: production
     destination:
       server: https://kubernetes.default.svc
       namespace: default
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
   ```

**Step 15: Configure Network Policies**
1. **Install Calico for network policies**:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico-vxlan.yaml
   ```

2. **Create network policies**:
   ```yaml
   # network-policy.yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: webapp-netpol
     namespace: default
   spec:
     podSelector:
       matchLabels:
         app: webapp
     policyTypes:
     - Ingress
     - Egress
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             name: ingress-nginx
       ports:
       - protocol: TCP
         port: 80
     egress:
     - to: []
       ports:
       - protocol: TCP
         port: 443
       - protocol: TCP
         port: 80
   ```

**Step 16: Backup and Disaster Recovery**
1. **Install Velero for backups**:
   ```bash
   wget https://github.com/vmware-tanzu/velero/releases/download/v1.9.0/velero-v1.9.0-linux-amd64.tar.gz
   tar -xvf velero-v1.9.0-linux-amd64.tar.gz
   sudo mv velero-v1.9.0-linux-amd64/velero /usr/local/bin/
   
   velero install \
     --provider aws \
     --plugins velero/velero-plugin-for-aws:v1.5.0 \
     --bucket eks-backup-bucket \
     --backup-location-config region=us-east-1 \
     --snapshot-location-config region=us-east-1
   ```

2. **Schedule regular backups**:
   ```bash
   velero schedule create daily-backup --schedule="0 2 * * *"
   ```

**Verification Checklist:**
- ✅ EKS cluster created with proper networking
- ✅ Node groups configured and running
- ✅ Essential add-ons installed (ALB Controller, Autoscaler)
- ✅ RBAC and security policies configured
- ✅ Sample application deployed and accessible
- ✅ Monitoring with Prometheus/Grafana working
- ✅ CloudWatch Container Insights enabled
- ✅ GitOps with ArgoCD configured
- ✅ Network policies implemented
- ✅ Backup and disaster recovery setup

### Task 45: Service Mesh with App Mesh
**Implement AWS App Mesh for microservices communication and observability.**

**Console-Based Solution:**

**Step 1: Navigate to App Mesh Console**
1. Go to **AWS App Mesh Console**
2. Click "Create mesh"

**Step 2: Create Service Mesh**
1. **Mesh configuration**:
   - **Mesh name**: "webapp-service-mesh"
   - **Egress filter**: ALLOW_ALL (for internet access)
   - **Tags**:
     - Environment: production
     - Application: webapp
     - Team: platform

**Step 3: Create Virtual Services**
1. **Create virtual service for user service**:
   - **Virtual service name**: "user-service.webapp.local"
   - **Provider**: Virtual node
   - **Virtual node**: user-service-vn (will create next)

2. **Create virtual service for order service**:
   - **Virtual service name**: "order-service.webapp.local"
   - **Provider**: Virtual router
   - **Virtual router**: order-service-vr (will create next)

**Step 4: Create Virtual Nodes**
1. **User service virtual node**:
   - **Virtual node name**: "user-service-vn"
   - **Service discovery**: DNS hostname
   - **DNS hostname**: user-service.default.svc.cluster.local
   - **Port**: 8080
   - **Protocol**: HTTP
   - **Health check**: Enable
     - **Path**: /health
     - **Interval**: 30 seconds
     - **Timeout**: 5 seconds
     - **Unhealthy threshold**: 3

2. **Order service virtual node**:
   - **Virtual node name**: "order-service-vn"
   - **Service discovery**: DNS hostname
   - **DNS hostname**: order-service.default.svc.cluster.local
   - **Port**: 8080
   - **Protocol**: HTTP
   - **Backends**: user-service.webapp.local

**Step 5: Create Virtual Router and Routes**
1. **Create virtual router**:
   - **Virtual router name**: "order-service-vr"
   - **Listener**: 
     - **Port**: 8080
     - **Protocol**: HTTP

2. **Create route for order service**:
   - **Route name**: "order-service-route"
   - **Route type**: HTTP
   - **Match**: 
     - **Prefix**: /
     - **Method**: ANY
   - **Action**:
     - **Target virtual node**: order-service-vn
     - **Weight**: 100

**Step 6: Deploy App Mesh Controller to EKS**
1. **Install App Mesh Controller**:
   ```bash
   # Add the EKS chart repo
   helm repo add eks https://aws.github.io/eks-charts
   helm repo update
   
   # Create namespace
   kubectl create ns appmesh-system
   
   # Install App Mesh Controller
   helm install appmesh-controller eks/appmesh-controller \
     --namespace appmesh-system \
     --set region=us-east-1 \
     --set serviceAccount.create=false \
     --set serviceAccount.name=appmesh-controller \
     --set log.level=info
   ```

2. **Create service account with IRSA**:
   ```bash
   # Create IAM role for service account
   eksctl create iamserviceaccount \
     --cluster=production-eks-cluster \
     --namespace=appmesh-system \
     --name=appmesh-controller \
     --attach-policy-arn=arn:aws:iam::aws:policy/AWSCloudMapFullAccess \
     --attach-policy-arn=arn:aws:iam::aws:policy/AWSAppMeshFullAccess \
     --override-existing-serviceaccounts \
     --approve
   ```

**Step 7: Create Kubernetes Resources with App Mesh**
1. **User service deployment**:
   ```yaml
   # user-service.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: user-service
     namespace: default
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: user-service
     template:
       metadata:
         labels:
           app: user-service
         annotations:
           appmesh.k8s.aws/mesh: webapp-service-mesh
       spec:
         containers:
         - name: user-service
           image: nginx:1.21
           ports:
           - containerPort: 8080
           env:
           - name: PORT
             value: "8080"
           resources:
             requests:
               memory: "128Mi"
               cpu: "100m"
             limits:
               memory: "256Mi"
               cpu: "200m"
           readinessProbe:
             httpGet:
               path: /health
               port: 8080
             initialDelaySeconds: 15
             periodSeconds: 10
           livenessProbe:
             httpGet:
               path: /health
               port: 8080
             initialDelaySeconds: 30
             periodSeconds: 30
   
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: user-service
     namespace: default
   spec:
     ports:
     - port: 8080
       targetPort: 8080
       protocol: TCP
     selector:
       app: user-service
   ```

2. **Order service deployment**:
   ```yaml
   # order-service.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: order-service
     namespace: default
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: order-service
     template:
       metadata:
         labels:
           app: order-service
         annotations:
           appmesh.k8s.aws/mesh: webapp-service-mesh
       spec:
         containers:
         - name: order-service
           image: nginx:1.21
           ports:
           - containerPort: 8080
           env:
           - name: PORT
             value: "8080"
           - name: USER_SERVICE_URL
             value: "http://user-service.webapp.local:8080"
           resources:
             requests:
               memory: "128Mi"
               cpu: "100m"
             limits:
               memory: "256Mi"
               cpu: "200m"
   
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: order-service
     namespace: default
   spec:
     ports:
     - port: 8080
       targetPort: 8080
       protocol: TCP
     selector:
       app: order-service
   ```

**Step 8: Configure Mesh Resources in Kubernetes**
1. **Create mesh CRD**:
   ```yaml
   # mesh.yaml
   apiVersion: appmesh.k8s.aws/v1beta2
   kind: Mesh
   metadata:
     name: webapp-service-mesh
   spec:
     namespaceSelector:
       matchLabels:
         mesh: webapp-service-mesh
     meshOwner: "123456789012"
   ```

2. **Create virtual node CRDs**:
   ```yaml
   # virtual-nodes.yaml
   apiVersion: appmesh.k8s.aws/v1beta2
   kind: VirtualNode
   metadata:
     name: user-service-vn
     namespace: default
   spec:
     awsName: user-service-vn
     podSelector:
       matchLabels:
         app: user-service
     listeners:
     - portMapping:
         port: 8080
         protocol: http
       healthCheck:
         protocol: http
         path: '/health'
         intervalMillis: 30000
         timeoutMillis: 5000
         unhealthyThreshold: 3
         healthyThreshold: 2
     serviceDiscovery:
       dns:
         hostname: user-service.default.svc.cluster.local
   
   ---
   apiVersion: appmesh.k8s.aws/v1beta2
   kind: VirtualNode
   metadata:
     name: order-service-vn
     namespace: default
   spec:
     awsName: order-service-vn
     podSelector:
       matchLabels:
         app: order-service
     listeners:
     - portMapping:
         port: 8080
         protocol: http
     backends:
     - virtualService:
         virtualServiceRef:
           name: user-service-vs
     serviceDiscovery:
       dns:
         hostname: order-service.default.svc.cluster.local
   ```

3. **Create virtual services CRDs**:
   ```yaml
   # virtual-services.yaml
   apiVersion: appmesh.k8s.aws/v1beta2
   kind: VirtualService
   metadata:
     name: user-service-vs
     namespace: default
   spec:
     awsName: user-service.webapp.local
     provider:
       virtualNode:
         virtualNodeRef:
           name: user-service-vn
   
   ---
   apiVersion: appmesh.k8s.aws/v1beta2
   kind: VirtualService
   metadata:
     name: order-service-vs
     namespace: default
   spec:
     awsName: order-service.webapp.local
     provider:
       virtualRouter:
         virtualRouterRef:
           name: order-service-vr
   ```

**Step 9: Configure Observability**
1. **Enable X-Ray tracing**:
   ```bash
   # Install X-Ray daemon
   kubectl apply -f https://raw.githubusercontent.com/aws-samples/aws-app-mesh-examples/master/examples/apps/djapp/1_base_application/xray-daemon.yaml
   ```

2. **Configure Envoy access logs**:
   ```yaml
   # Update virtual nodes with logging
   spec:
     logging:
       accessLog:
         file:
           path: /dev/stdout
   ```

**Step 10: Implement Traffic Management**
1. **Blue/Green deployment with virtual router**:
   ```yaml
   # blue-green-route.yaml
   apiVersion: appmesh.k8s.aws/v1beta2
   kind: VirtualRouter
   metadata:
     name: order-service-vr
     namespace: default
   spec:
     awsName: order-service-vr
     listeners:
     - portMapping:
         port: 8080
         protocol: http
     routes:
     - name: order-service-route
       httpRoute:
         match:
           prefix: /
         action:
           weightedTargets:
           - virtualNodeRef:
               name: order-service-blue-vn
             weight: 80
           - virtualNodeRef:
               name: order-service-green-vn
             weight: 20
   ```

2. **Canary deployment configuration**:
   ```yaml
   # canary-route.yaml
   apiVersion: appmesh.k8s.aws/v1beta2
   kind: VirtualRouter
   metadata:
     name: user-service-vr
     namespace: default
   spec:
     listeners:
     - portMapping:
         port: 8080
         protocol: http
     routes:
     - name: canary-route
       httpRoute:
         match:
           prefix: /
           headers:
           - name: canary
             match:
               exact: "true"
         action:
           weightedTargets:
           - virtualNodeRef:
               name: user-service-canary-vn
             weight: 100
     - name: stable-route
       httpRoute:
         match:
           prefix: /
         action:
           weightedTargets:
           - virtualNodeRef:
               name: user-service-stable-vn
             weight: 100
   ```

**Step 11: Configure Circuit Breaker**
1. **Add outlier detection**:
   ```yaml
   # Add to virtual node spec
   spec:
     listeners:
     - portMapping:
         port: 8080
         protocol: http
       outlierDetection:
         maxServerErrors: 5
         intervalMillis: 30000
         baseEjectionDurationMillis: 15000
         maxEjectionPercent: 50
   ```

**Step 12: Monitor Service Mesh**
1. **CloudWatch metrics**:
   - Request volume and latency
   - Error rates by service
   - Circuit breaker status
   - Health check status

2. **X-Ray service map**:
   - End-to-end request tracing
   - Performance bottleneck identification
   - Error root cause analysis

**Step 13: Implement Security Policies**
1. **TLS encryption between services**:
   ```yaml
   # Add to virtual node listeners
   spec:
     listeners:
     - portMapping:
         port: 8080
         protocol: http
       tls:
         mode: STRICT
         certificate:
           acm:
             certificateArn: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
   ```

2. **Backend defaults with TLS**:
   ```yaml
   # Add to virtual node spec
   spec:
     backendDefaults:
       clientPolicy:
         tls:
           enforce: true
           ports: [8080]
           validation:
             trust:
               acm:
                 certificateAuthorityArns:
                 - arn:aws:acm-pca:us-east-1:123456789012:certificate-authority/12345678-1234-1234-1234-123456789012
   ```

**Verification Checklist:**
- ✅ App Mesh created with proper configuration
- ✅ Virtual services, nodes, and routers configured
- ✅ App Mesh controller deployed to EKS
- ✅ Microservices deployed with mesh integration
- ✅ Traffic routing and load balancing working
- ✅ Observability with X-Ray and CloudWatch enabled
- ✅ Circuit breaker and outlier detection configured
- ✅ Blue/Green and canary deployments tested
- ✅ TLS encryption between services enabled
- ✅ Service mesh monitoring dashboard created

### Task 46: Data Pipeline with Glue and Athena
**Build a serverless data pipeline for ETL processing and analytics.**

**Console-Based Solution:**

**Step 1: Create S3 Data Lake Structure**
1. **Create S3 buckets**:
   - **Raw data bucket**: "company-datalake-raw-us-east-1"
   - **Processed data bucket**: "company-datalake-processed-us-east-1"
   - **Analytics bucket**: "company-datalake-analytics-us-east-1"
   - **Glue scripts bucket**: "company-glue-scripts-us-east-1"

2. **Set up folder structure**:
   ```
   company-datalake-raw/
   ├── sales/
   │   ├── year=2024/month=01/day=01/
   │   └── year=2024/month=01/day=02/
   ├── customers/
   │   └── year=2024/month=01/day=01/
   └── products/
       └── year=2024/month=01/day=01/
   
   company-datalake-processed/
   ├── sales_cleaned/
   ├── customer_enriched/
   └── product_catalog/
   
   company-datalake-analytics/
   ├── daily_sales_summary/
   ├── customer_analytics/
   └── product_performance/
   ```

**Step 2: Set Up AWS Glue Data Catalog**
1. **Navigate to AWS Glue Console**
2. **Create database**:
   - **Database name**: "company_datalake"
   - **Description**: "Main data lake database for analytics"

3. **Create crawlers for each data source**:
   - **Sales crawler**:
     - **Name**: "sales-data-crawler"
     - **Data source**: S3 path `s3://company-datalake-raw/sales/`
     - **IAM role**: Create new role with S3 access
     - **Database**: company_datalake
     - **Table prefix**: raw_
     - **Schedule**: Daily at 6:00 AM UTC

**Step 3: Create Sample Data Files**
1. **Sales data (CSV format)**:
   ```csv
   # s3://company-datalake-raw/sales/year=2024/month=01/day=01/sales_20240101.csv
   transaction_id,customer_id,product_id,quantity,unit_price,transaction_date,store_id
   TXN_001,CUST_123,PROD_456,2,29.99,2024-01-01 10:30:00,STORE_001
   TXN_002,CUST_124,PROD_457,1,49.99,2024-01-01 11:15:00,STORE_001
   TXN_003,CUST_125,PROD_458,3,19.99,2024-01-01 12:00:00,STORE_002
   ```

2. **Customer data (JSON format)**:
   ```json
   # s3://company-datalake-raw/customers/year=2024/month=01/day=01/customers_20240101.json
   {"customer_id": "CUST_123", "first_name": "John", "last_name": "Doe", "email": "john.doe@email.com", "registration_date": "2023-06-15", "customer_segment": "Premium"}
   {"customer_id": "CUST_124", "first_name": "Jane", "last_name": "Smith", "email": "jane.smith@email.com", "registration_date": "2023-08-20", "customer_segment": "Standard"}
   {"customer_id": "CUST_125", "first_name": "Bob", "last_name": "Johnson", "email": "bob.johnson@email.com", "registration_date": "2023-12-01", "customer_segment": "Premium"}
   ```

**Step 4: Create ETL Jobs with AWS Glue**
1. **Create ETL job for sales data cleaning**:
   - **Job name**: "sales-data-etl"
   - **Type**: Spark
   - **Glue version**: 4.0
   - **Language**: Python 3
   - **Worker type**: G.1X
   - **Number of workers**: 2
   - **Job timeout**: 2880 minutes

2. **ETL script for sales data**:
   ```python
   # sales_etl.py
   import sys
   from awsglue.transforms import *
   from awsglue.utils import getResolvedOptions
   from pyspark.context import SparkContext
   from awsglue.context import GlueContext
   from awsglue.job import Job
   from pyspark.sql import functions as F
   from pyspark.sql.types import *
   
   args = getResolvedOptions(sys.argv, ['JOB_NAME', 'SOURCE_DATABASE', 'TARGET_BUCKET'])
   
   sc = SparkContext()
   glueContext = GlueContext(sc)
   spark = glueContext.spark_session
   job = Job(glueContext)
   job.init(args['JOB_NAME'], args)
   
   # Read sales data from catalog
   sales_dyf = glueContext.create_dynamic_frame.from_catalog(
       database = args['SOURCE_DATABASE'],
       table_name = "raw_sales"
   )
   
   # Convert to DataFrame for complex transformations
   sales_df = sales_dyf.toDF()
   
   # Data cleaning and transformations
   sales_cleaned = sales_df \
       .filter(F.col("quantity") > 0) \
       .filter(F.col("unit_price") > 0) \
       .withColumn("total_amount", F.col("quantity") * F.col("unit_price")) \
       .withColumn("transaction_date", F.to_timestamp("transaction_date", "yyyy-MM-dd HH:mm:ss")) \
       .withColumn("year", F.year("transaction_date")) \
       .withColumn("month", F.month("transaction_date")) \
       .withColumn("day", F.dayofmonth("transaction_date")) \
       .withColumn("hour", F.hour("transaction_date"))
   
   # Data quality checks
   total_records = sales_df.count()
   cleaned_records = sales_cleaned.count()
   
   print(f"Original records: {total_records}")
   print(f"Cleaned records: {cleaned_records}")
   print(f"Records removed: {total_records - cleaned_records}")
   
   # Convert back to DynamicFrame
   sales_dyf_cleaned = DynamicFrame.fromDF(sales_cleaned, glueContext, "sales_cleaned")
   
   # Write to S3 in Parquet format
   glueContext.write_dynamic_frame.from_options(
       frame = sales_dyf_cleaned,
       connection_type = "s3",
       connection_options = {
           "path": f"s3://{args['TARGET_BUCKET']}/sales_cleaned/",
           "partitionKeys": ["year", "month", "day"]
       },
       format = "parquet"
   )
   
   job.commit()
   ```

**Step 5: Create Data Quality Jobs**
1. **Create data quality ruleset**:
   ```python
   # data_quality_sales.py
   import sys
   from awsglue.transforms import *
   from awsglue.utils import getResolvedOptions
   from pyspark.context import SparkContext
   from awsglue.context import GlueContext
   from awsglue.job import Job
   from pyspark.sql import functions as F
   
   def validate_sales_data(df):
       """Data quality validation rules"""
       
       # Rule 1: No null values in critical fields
       null_checks = {
           'transaction_id': df.filter(F.col("transaction_id").isNull()).count(),
           'customer_id': df.filter(F.col("customer_id").isNull()).count(),
           'product_id': df.filter(F.col("product_id").isNull()).count(),
           'quantity': df.filter(F.col("quantity").isNull()).count(),
           'unit_price': df.filter(F.col("unit_price").isNull()).count()
       }
       
       # Rule 2: Positive values for quantity and price
       negative_quantity = df.filter(F.col("quantity") <= 0).count()
       negative_price = df.filter(F.col("unit_price") <= 0).count()
       
       # Rule 3: Valid date ranges
       invalid_dates = df.filter(
           (F.col("transaction_date") < "2020-01-01") | 
           (F.col("transaction_date") > F.current_date())
       ).count()
       
       # Rule 4: Duplicate transaction IDs
       total_records = df.count()
       unique_transactions = df.select("transaction_id").distinct().count()
       duplicates = total_records - unique_transactions
       
       # Generate quality report
       quality_report = {
           'total_records': total_records,
           'null_values': null_checks,
           'negative_quantity': negative_quantity,
           'negative_price': negative_price,
           'invalid_dates': invalid_dates,
           'duplicate_transactions': duplicates,
           'data_quality_score': calculate_quality_score(
               total_records, null_checks, negative_quantity, 
               negative_price, invalid_dates, duplicates
           )
       }
       
       return quality_report
   
   def calculate_quality_score(total, nulls, neg_qty, neg_price, inv_dates, dups):
       """Calculate overall data quality score"""
       issues = sum(nulls.values()) + neg_qty + neg_price + inv_dates + dups
       if total == 0:
           return 0
       return max(0, (total - issues) / total * 100)
   ```

**Step 6: Set Up Amazon Athena**
1. **Configure Athena query result location**:
   - **S3 bucket**: s3://company-athena-results-us-east-1/
   - **Encryption**: SSE-S3

2. **Create external tables in Athena**:
   ```sql
   -- Create table for cleaned sales data
   CREATE EXTERNAL TABLE IF NOT EXISTS company_datalake.sales_cleaned (
       transaction_id string,
       customer_id string,
       product_id string,
       quantity int,
       unit_price decimal(10,2),
       total_amount decimal(10,2),
       transaction_date timestamp,
       store_id string,
       hour int
   )
   PARTITIONED BY (
       year int,
       month int,
       day int
   )
   STORED AS PARQUET
   LOCATION 's3://company-datalake-processed/sales_cleaned/'
   TBLPROPERTIES ('has_encrypted_data'='false');
   
   -- Repair partitions
   MSCK REPAIR TABLE company_datalake.sales_cleaned;
   ```

**Step 7: Create Analytics Queries**
1. **Daily sales summary**:
   ```sql
   CREATE TABLE company_datalake.daily_sales_summary AS
   WITH daily_metrics AS (
       SELECT 
           year,
           month,
           day,
           COUNT(*) as transaction_count,
           SUM(total_amount) as total_revenue,
           AVG(total_amount) as avg_transaction_value,
           COUNT(DISTINCT customer_id) as unique_customers,
           COUNT(DISTINCT product_id) as unique_products,
           MIN(total_amount) as min_transaction,
           MAX(total_amount) as max_transaction
       FROM company_datalake.sales_cleaned
       GROUP BY year, month, day
   )
   SELECT 
       CONCAT(CAST(year AS varchar), '-', 
              LPAD(CAST(month AS varchar), 2, '0'), '-', 
              LPAD(CAST(day AS varchar), 2, '0')) as date,
       transaction_count,
       ROUND(total_revenue, 2) as total_revenue,
       ROUND(avg_transaction_value, 2) as avg_transaction_value,
       unique_customers,
       unique_products,
       ROUND(total_revenue / unique_customers, 2) as revenue_per_customer,
       min_transaction,
       max_transaction
   FROM daily_metrics
   ORDER BY year, month, day;
   ```

2. **Customer segmentation analysis**:
   ```sql
   CREATE TABLE company_datalake.customer_analytics AS
   WITH customer_metrics AS (
       SELECT 
           s.customer_id,
           COUNT(*) as total_transactions,
           SUM(s.total_amount) as total_spent,
           AVG(s.total_amount) as avg_transaction_value,
           MIN(s.transaction_date) as first_purchase,
           MAX(s.transaction_date) as last_purchase,
           COUNT(DISTINCT s.product_id) as unique_products_purchased,
           COUNT(DISTINCT s.store_id) as stores_visited
       FROM company_datalake.sales_cleaned s
       GROUP BY s.customer_id
   )
   SELECT 
       cm.*,
       CASE 
           WHEN total_spent >= 1000 AND total_transactions >= 10 THEN 'VIP'
           WHEN total_spent >= 500 AND total_transactions >= 5 THEN 'High Value'
           WHEN total_spent >= 200 AND total_transactions >= 3 THEN 'Regular'
           ELSE 'Occasional'
       END as customer_tier,
       DATE_DIFF('day', date(last_purchase), CURRENT_DATE) as days_since_last_purchase
   FROM customer_metrics
   ORDER BY total_spent DESC;
   ```

**Step 8: Create Data Pipeline Workflow**
1. **Create Glue workflow**:
   - **Workflow name**: "daily-data-pipeline"
   - **Description**: "Daily ETL pipeline for sales data processing"

2. **Add triggers**:
   - **Schedule trigger**: Daily at 7:00 AM UTC
   - **Conditional trigger**: Run data quality after ETL completion

3. **Add jobs to workflow**:
   - **Crawler**: sales-data-crawler
   - **ETL Job**: sales-data-etl
   - **Data Quality Job**: sales-data-quality
   - **Analytics Job**: generate-daily-reports

**Step 9: Implement Real-time Data Ingestion**
1. **Create Kinesis Data Firehose**:
   - **Delivery stream name**: "sales-realtime-stream"
   - **Source**: Direct PUT or other sources
   - **Destination**: S3 bucket (raw data)
   - **Buffer size**: 128 MB or 900 seconds
   - **Compression**: GZIP
   - **Data transformation**: Enable with Lambda function

2. **Lambda function for data transformation**:
   ```python
   import json
   import base64
   import boto3
   from datetime import datetime
   
   def lambda_handler(event, context):
       output = []
       
       for record in event['records']:
           # Decode the data
           payload = base64.b64decode(record['data'])
           data = json.loads(payload)
           
           # Transform the data
           transformed_data = {
               'transaction_id': data.get('transaction_id'),
               'customer_id': data.get('customer_id'),
               'product_id': data.get('product_id'),
               'quantity': int(data.get('quantity', 0)),
               'unit_price': float(data.get('unit_price', 0.0)),
               'transaction_date': data.get('transaction_date', datetime.utcnow().isoformat()),
               'store_id': data.get('store_id'),
               'total_amount': int(data.get('quantity', 0)) * float(data.get('unit_price', 0.0)),
               'processed_at': datetime.utcnow().isoformat()
           }
           
           # Encode the transformed data
           output_record = {
               'recordId': record['recordId'],
               'result': 'Ok',
               'data': base64.b64encode(
                   json.dumps(transformed_data).encode('utf-8')
               ).decode('utf-8')
           }
           output.append(output_record)
       
       return {'records': output}
   ```

**Step 10: Set Up Monitoring and Alerting**
1. **CloudWatch metrics for Glue jobs**:
   - Job success/failure rates
   - Job duration and performance
   - Data quality scores
   - Record counts processed

2. **Create CloudWatch alarms**:
   ```python
   # CloudWatch alarm for job failures
   import boto3
   
   cloudwatch = boto3.client('cloudwatch')
   
   cloudwatch.put_metric_alarm(
       AlarmName='GlueJobFailure-SalesETL',
       ComparisonOperator='GreaterThanThreshold',
       EvaluationPeriods=1,
       MetricName='glue.driver.aggregate.numFailedTasks',
       Namespace='Glue',
       Period=300,
       Statistic='Sum',
       Threshold=0.0,
       ActionsEnabled=True,
       AlarmActions=[
           'arn:aws:sns:us-east-1:123456789012:data-pipeline-alerts'
       ],
       AlarmDescription='Alert when Glue ETL job fails',
       Dimensions=[
           {
               'Name': 'JobName',
               'Value': 'sales-data-etl'
           },
       ]
   )
   ```

**Step 11: Cost Optimization**
1. **Implement lifecycle policies**:
   ```json
   {
     "Rules": [
       {
         "ID": "DataLakeLifecycle",
         "Status": "Enabled",
         "Filter": {"Prefix": "sales/"},
         "Transitions": [
           {
             "Days": 30,
             "StorageClass": "STANDARD_IA"
           },
           {
             "Days": 90,
             "StorageClass": "GLACIER"
           },
           {
             "Days": 365,
             "StorageClass": "DEEP_ARCHIVE"
           }
         ]
       }
     ]
   }
   ```

2. **Optimize Glue job resources**:
   - Use appropriate worker types
   - Enable job bookmarks
   - Partition data for efficient querying

**Verification Checklist:**
- ✅ S3 data lake structure created
- ✅ Glue Data Catalog configured with crawlers
- ✅ ETL jobs processing data successfully
- ✅ Data quality validation implemented
- ✅ Athena tables created and queryable
- ✅ Analytics queries producing insights
- ✅ Real-time data ingestion with Kinesis
- ✅ Workflow orchestration with Glue
- ✅ Monitoring and alerting configured
- ✅ Cost optimization measures implemented

### Task 47: Machine Learning Pipeline
**Build an end-to-end ML pipeline using SageMaker for model training and deployment.**

**Console-Based Solution:**

**Step 1: Set Up SageMaker Environment**
1. **Navigate to Amazon SageMaker Console**
2. **Create notebook instance**:
   - **Notebook instance name**: "ml-development-notebook"
   - **Instance type**: ml.t3.medium (for development)
   - **IAM role**: Create new role with S3 access
   - **Git repositories**: Optional - clone your ML code repository

3. **Create S3 bucket for ML artifacts**:
   - **Bucket name**: "company-ml-pipeline-us-east-1"
   - **Folder structure**:
     ```
     company-ml-pipeline/
     ├── data/
     │   ├── raw/
     │   ├── processed/
     │   └── features/
     ├── models/
     │   ├── training/
     │   └── artifacts/
     ├── scripts/
     └── outputs/
     ```

**Step 2: Prepare Training Data**
1. **Sample customer churn dataset**:
   ```python
   # data_preparation.py
   import pandas as pd
   import numpy as np
   from sklearn.model_selection import train_test_split
   from sklearn.preprocessing import StandardScaler, LabelEncoder
   import boto3
   
   def create_sample_data():
       """Create sample customer churn dataset"""
       np.random.seed(42)
       n_customers = 10000
       
       # Generate synthetic customer data
       data = {
           'customer_id': [f'CUST_{i:06d}' for i in range(n_customers)],
           'age': np.random.normal(40, 15, n_customers).astype(int),
           'tenure_months': np.random.exponential(24, n_customers).astype(int),
           'monthly_charges': np.random.normal(65, 20, n_customers),
           'total_charges': np.random.normal(2000, 1500, n_customers),
           'contract_type': np.random.choice(['Month-to-month', 'One year', 'Two year'], n_customers),
           'payment_method': np.random.choice(['Electronic check', 'Mailed check', 'Bank transfer', 'Credit card'], n_customers),
           'internet_service': np.random.choice(['DSL', 'Fiber optic', 'No'], n_customers),
           'online_security': np.random.choice(['Yes', 'No'], n_customers),
           'tech_support': np.random.choice(['Yes', 'No'], n_customers),
           'streaming_tv': np.random.choice(['Yes', 'No'], n_customers),
           'paperless_billing': np.random.choice(['Yes', 'No'], n_customers)
       }
       
       df = pd.DataFrame(data)
       
       # Create churn target based on business logic
       churn_probability = (
           0.1 +  # Base churn rate
           (df['contract_type'] == 'Month-to-month') * 0.3 +
           (df['tenure_months'] < 12) * 0.2 +
           (df['monthly_charges'] > 80) * 0.15 +
           (df['tech_support'] == 'No') * 0.1 +
           np.random.normal(0, 0.1, n_customers)
       )
       
       df['churn'] = (np.random.random(n_customers) < churn_probability).astype(int)
       
       return df
   
   def preprocess_data(df):
       """Preprocess data for ML training"""
       # Handle missing values
       df = df.fillna(df.median(numeric_only=True))
       
       # Encode categorical variables
       categorical_columns = ['contract_type', 'payment_method', 'internet_service', 
                            'online_security', 'tech_support', 'streaming_tv', 'paperless_billing']
       
       for col in categorical_columns:
           le = LabelEncoder()
           df[col] = le.fit_transform(df[col])
       
       # Feature engineering
       df['avg_monthly_charges'] = df['total_charges'] / (df['tenure_months'] + 1)
       df['charges_per_service'] = df['monthly_charges'] / (
           df[['online_security', 'tech_support', 'streaming_tv']].sum(axis=1) + 1
       )
       
       # Remove customer_id for training
       features = df.drop(['customer_id', 'churn'], axis=1)
       target = df['churn']
       
       return features, target
   
   # Create and upload data
   df = create_sample_data()
   features, target = preprocess_data(df)
   
   # Split data
   X_train, X_test, y_train, y_test = train_test_split(
       features, target, test_size=0.2, random_state=42, stratify=target
   )
   
   # Save to S3
   train_data = pd.concat([y_train, X_train], axis=1)
   test_data = pd.concat([y_test, X_test], axis=1)
   
   train_data.to_csv('s3://company-ml-pipeline-us-east-1/data/processed/train.csv', index=False)
   test_data.to_csv('s3://company-ml-pipeline-us-east-1/data/processed/test.csv', index=False)
   ```

**Step 3: Create Training Script**
1. **XGBoost training script**:
   ```python
   # training/train.py
   import argparse
   import os
   import pandas as pd
   import xgboost as xgb
   from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, roc_auc_score
   import joblib
   import json
   import boto3
   
   def parse_args():
       parser = argparse.ArgumentParser()
       
       # SageMaker specific arguments
       parser.add_argument('--model-dir', type=str, default=os.environ.get('SM_MODEL_DIR'))
       parser.add_argument('--train', type=str, default=os.environ.get('SM_CHANNEL_TRAIN'))
       parser.add_argument('--validation', type=str, default=os.environ.get('SM_CHANNEL_VALIDATION'))
       
       # Hyperparameters
       parser.add_argument('--max-depth', type=int, default=6)
       parser.add_argument('--eta', type=float, default=0.3)
       parser.add_argument('--gamma', type=float, default=0)
       parser.add_argument('--min-child-weight', type=int, default=1)
       parser.add_argument('--subsample', type=float, default=1)
       parser.add_argument('--n-estimators', type=int, default=100)
       
       return parser.parse_args()
   
   def load_data(path):
       """Load data from CSV file"""
       data = pd.read_csv(f"{path}/train.csv")
       y = data.iloc[:, 0]  # First column is target
       X = data.iloc[:, 1:]  # Rest are features
       return X, y
   
   def train_model(X_train, y_train, X_val, y_val, args):
       """Train XGBoost model"""
       model = xgb.XGBClassifier(
           max_depth=args.max_depth,
           eta=args.eta,
           gamma=args.gamma,
           min_child_weight=args.min_child_weight,
           subsample=args.subsample,
           n_estimators=args.n_estimators,
           random_state=42
       )
       
       # Train model
       model.fit(
           X_train, y_train,
           eval_set=[(X_val, y_val)],
           early_stopping_rounds=10,
           verbose=False
       )
       
       return model
   
   def evaluate_model(model, X_test, y_test):
       """Evaluate model performance"""
       y_pred = model.predict(X_test)
       y_pred_proba = model.predict_proba(X_test)[:, 1]
       
       metrics = {
           'accuracy': accuracy_score(y_test, y_pred),
           'precision': precision_score(y_test, y_pred),
           'recall': recall_score(y_test, y_pred),
           'f1_score': f1_score(y_test, y_pred),
           'roc_auc': roc_auc_score(y_test, y_pred_proba)
       }
       
       return metrics
   
   def save_model(model, model_dir):
       """Save model artifacts"""
       # Save model
       joblib.dump(model, os.path.join(model_dir, 'model.joblib'))
       
       # Save feature names
       feature_names = model.feature_names_in_
       with open(os.path.join(model_dir, 'feature_names.json'), 'w') as f:
           json.dump(feature_names.tolist(), f)
   
   if __name__ == '__main__':
       args = parse_args()
       
       # Load data
       X_train, y_train = load_data(args.train)
       X_val, y_val = load_data(args.validation)
       
       print(f"Training data shape: {X_train.shape}")
       print(f"Validation data shape: {X_val.shape}")
       
       # Train model
       model = train_model(X_train, y_train, X_val, y_val, args)
       
       # Evaluate model
       metrics = evaluate_model(model, X_val, y_val)
       print(f"Model performance: {metrics}")
       
       # Save model
       save_model(model, args.model_dir)
       print("Model saved successfully")
   ```

**Step 4: Create SageMaker Training Job**
1. **Training job configuration**:
   ```python
   # create_training_job.py
   import sagemaker
   from sagemaker.sklearn.estimator import SKLearn
   from sagemaker.tuner import HyperparameterTuner, IntegerParameter, ContinuousParameter
   import boto3
   
   # Initialize SageMaker session
   sagemaker_session = sagemaker.Session()
   role = sagemaker.get_execution_role()
   
   # Define training job
   sklearn_estimator = SKLearn(
       entry_point='train.py',
       source_dir='training',
       framework_version='1.0-1',
       instance_type='ml.m5.large',
       instance_count=1,
       role=role,
       sagemaker_session=sagemaker_session,
       hyperparameters={
           'max-depth': 6,
           'eta': 0.3,
           'n-estimators': 100
       }
   )
   
   # Define input data channels
   train_input = sagemaker.TrainingInput(
       's3://company-ml-pipeline-us-east-1/data/processed/train.csv',
       content_type='text/csv'
   )
   
   validation_input = sagemaker.TrainingInput(
       's3://company-ml-pipeline-us-east-1/data/processed/test.csv',
       content_type='text/csv'
   )
   
   # Start training job
   sklearn_estimator.fit({
       'train': train_input,
       'validation': validation_input
   })
   ```

**Step 5: Hyperparameter Tuning**
1. **Create hyperparameter tuning job**:
   ```python
   # hyperparameter_tuning.py
   from sagemaker.tuner import HyperparameterTuner, IntegerParameter, ContinuousParameter
   
   # Define hyperparameter ranges
   hyperparameter_ranges = {
       'max-depth': IntegerParameter(3, 10),
       'eta': ContinuousParameter(0.1, 0.5),
       'gamma': ContinuousParameter(0, 0.5),
       'min-child-weight': IntegerParameter(1, 10),
       'subsample': ContinuousParameter(0.8, 1.0),
       'n-estimators': IntegerParameter(50, 200)
   }
   
   # Define objective metric
   objective_metric_name = 'validation:auc'
   objective_type = 'Maximize'
   
   # Create tuner
   tuner = HyperparameterTuner(
       sklearn_estimator,
       objective_metric_name,
       hyperparameter_ranges,
       objective_type=objective_type,
       max_jobs=20,
       max_parallel_jobs=3
   )
   
   # Start tuning job
   tuner.fit({
       'train': train_input,
       'validation': validation_input
   })
   
   # Get best training job
   best_training_job = tuner.best_training_job()
   print(f"Best training job: {best_training_job}")
   ```

**Step 6: Model Registry and Versioning**
1. **Register model in SageMaker Model Registry**:
   ```python
   # model_registry.py
   from sagemaker.model import Model
   from sagemaker.model_package import ModelPackage
   
   # Create model from training artifacts
   model = Model(
       image_uri=sklearn_estimator.image_uri,
       model_data=sklearn_estimator.model_data,
       role=role,
       predictor_cls=sagemaker.predictor.Predictor
   )
   
   # Register model package
   model_package = model.register(
       content_types=["text/csv"],
       response_types=["text/csv"],
       inference_instances=["ml.t2.medium", "ml.m5.large"],
       transform_instances=["ml.m5.large"],
       model_package_group_name="churn-prediction-models",
       approval_status="PendingManualApproval",
       description="Customer churn prediction model v1.0"
   )
   ```

**Step 7: Create Model Endpoint**
1. **Deploy model to endpoint**:
   ```python
   # model_deployment.py
   from sagemaker.sklearn.model import SKLearnModel
   from sagemaker.serializers import CSVSerializer
   from sagemaker.deserializers import CSVDeserializer
   
   # Create model from artifacts
   sklearn_model = SKLearnModel(
       model_data=sklearn_estimator.model_data,
       role=role,
       entry_point='inference.py',
       source_dir='inference',
       framework_version='1.0-1'
   )
   
   # Deploy model
   predictor = sklearn_model.deploy(
       initial_instance_count=1,
       instance_type='ml.t2.medium',
       endpoint_name='churn-prediction-endpoint',
       serializer=CSVSerializer(),
       deserializer=CSVDeserializer()
   )
   ```

2. **Inference script**:
   ```python
   # inference/inference.py
   import joblib
   import json
   import pandas as pd
   import numpy as np
   import os
   
   def model_fn(model_dir):
       """Load model for inference"""
       model = joblib.load(os.path.join(model_dir, 'model.joblib'))
       return model
   
   def input_fn(request_body, request_content_type):
       """Parse input data"""
       if request_content_type == 'text/csv':
           # Parse CSV input
           df = pd.read_csv(io.StringIO(request_body))
           return df
       elif request_content_type == 'application/json':
           # Parse JSON input
           data = json.loads(request_body)
           df = pd.DataFrame(data)
           return df
       else:
           raise ValueError(f"Unsupported content type: {request_content_type}")
   
   def predict_fn(input_data, model):
       """Make predictions"""
       predictions = model.predict_proba(input_data)
       
       # Return both class predictions and probabilities
       result = {
           'predictions': model.predict(input_data).tolist(),
           'probabilities': predictions[:, 1].tolist(),  # Probability of churn
           'confidence': np.max(predictions, axis=1).tolist()
       }
       
       return result
   
   def output_fn(prediction, content_type):
       """Format output"""
       if content_type == 'application/json':
           return json.dumps(prediction)
       elif content_type == 'text/csv':
           # Return CSV format
           df = pd.DataFrame(prediction)
           return df.to_csv(index=False)
       else:
           raise ValueError(f"Unsupported content type: {content_type}")
   ```

**Step 8: Create Batch Transform Job**
1. **Batch inference for large datasets**:
   ```python
   # batch_transform.py
   from sagemaker.sklearn.model import SKLearnModel
   from sagemaker.transformer import Transformer
   
   # Create transformer
   transformer = sklearn_model.transformer(
       instance_count=1,
       instance_type='ml.m5.large',
       output_path='s3://company-ml-pipeline-us-east-1/outputs/batch-predictions/',
       accept='text/csv'
   )
   
   # Start batch transform job
   transformer.transform(
       's3://company-ml-pipeline-us-east-1/data/processed/batch_input.csv',
       content_type='text/csv',
       split_type='Line'
   )
   
   # Wait for completion
   transformer.wait()
   ```

**Step 9: Model Monitoring**
1. **Set up model monitoring**:
   ```python
   # model_monitoring.py
   from sagemaker.model_monitor import DefaultModelMonitor
   from sagemaker.model_monitor.dataset_format import DatasetFormat
   
   # Create model monitor
   model_monitor = DefaultModelMonitor(
       role=role,
       instance_count=1,
       instance_type='ml.m5.xlarge',
       volume_size_in_gb=20,
       max_runtime_in_seconds=3600
   )
   
   # Create baseline
   baseline_job = model_monitor.suggest_baseline(
       baseline_dataset='s3://company-ml-pipeline-us-east-1/data/processed/train.csv',
       dataset_format=DatasetFormat.csv(header=True),
       output_s3_uri='s3://company-ml-pipeline-us-east-1/monitoring/baseline'
   )
   
   # Enable monitoring
   predictor.update_data_capture_config(
       data_capture_config=DataCaptureConfig(
           enable_capture=True,
           sampling_percentage=100,
           destination_s3_uri='s3://company-ml-pipeline-us-east-1/monitoring/data-capture'
       )
   )
   
   # Create monitoring schedule
   model_monitor.create_monitoring_schedule(
       monitor_schedule_name='churn-model-monitoring',
       endpoint_input=predictor.endpoint_name,
       output_s3_uri='s3://company-ml-pipeline-us-east-1/monitoring/results',
       statistics=baseline_job.baseline_statistics(),
       constraints=baseline_job.suggested_constraints(),
       schedule_cron_expression='cron(0 20 ? * MON-FRI *)'  # Daily at 8 PM
   )
   ```

**Step 10: A/B Testing Framework**
1. **Multi-variant endpoint**:
   ```python
   # ab_testing.py
   from sagemaker.multidatamodel import MultiDataModel
   
   # Create multi-model endpoint
   multi_model = MultiDataModel(
       name="churn-prediction-multi-model",
       model_data_prefix='s3://company-ml-pipeline-us-east-1/models/',
       image_uri=sklearn_estimator.image_uri,
       role=role
   )
   
   # Deploy with multiple model variants
   predictor = multi_model.deploy(
       initial_instance_count=1,
       instance_type='ml.m5.large',
       endpoint_name='churn-prediction-ab-test'
   )
   
   # A/B test configuration
   ab_test_config = {
       'model_a': {
           'weight': 70,  # 70% of traffic
           'model_name': 'churn-model-v1.tar.gz'
       },
       'model_b': {
           'weight': 30,  # 30% of traffic
           'model_name': 'churn-model-v2.tar.gz'
       }
   }
   ```

**Step 11: MLOps Pipeline with SageMaker Pipelines**
1. **Create ML pipeline**:
   ```python
   # mlops_pipeline.py
   from sagemaker.workflow.pipeline import Pipeline
   from sagemaker.workflow.steps import TrainingStep, CreateModelStep, RegisterModel
   from sagemaker.workflow.parameters import ParameterString, ParameterFloat
   
   # Define pipeline parameters
   processing_instance_count = ParameterInteger(name="ProcessingInstanceCount", default_value=1)
   training_instance_type = ParameterString(name="TrainingInstanceType", default_value="ml.m5.large")
   model_approval_status = ParameterString(name="ModelApprovalStatus", default_value="PendingManualApproval")
   
   # Data processing step
   processing_step = ProcessingStep(
       name="DataProcessing",
       processor=sklearn_processor,
       inputs=[
           ProcessingInput(source=input_data, destination="/opt/ml/processing/input"),
       ],
       outputs=[
           ProcessingOutput(output_name="train", source="/opt/ml/processing/train"),
           ProcessingOutput(output_name="validation", source="/opt/ml/processing/validation"),
           ProcessingOutput(output_name="test", source="/opt/ml/processing/test"),
       ],
       code="preprocessing.py",
   )
   
   # Training step
   training_step = TrainingStep(
       name="TrainChurnModel",
       estimator=sklearn_estimator,
       inputs={
           "train": TrainingInput(
               s3_data=processing_step.properties.ProcessingOutputConfig.Outputs["train"].S3Output.S3Uri,
               content_type="text/csv"
           ),
           "validation": TrainingInput(
               s3_data=processing_step.properties.ProcessingOutputConfig.Outputs["validation"].S3Output.S3Uri,
               content_type="text/csv"
           ),
       },
   )
   
   # Model registration step
   register_step = RegisterModel(
       name="RegisterChurnModel",
       estimator=sklearn_estimator,
       model_data=training_step.properties.ModelArtifacts.S3ModelArtifacts,
       content_types=["text/csv"],
       response_types=["text/csv"],
       inference_instances=["ml.t2.medium", "ml.m5.large"],
       transform_instances=["ml.m5.large"],
       model_package_group_name="churn-prediction-models",
       approval_status=model_approval_status,
   )
   
   # Create pipeline
   pipeline = Pipeline(
       name="ChurnPredictionPipeline",
       parameters=[
           processing_instance_count,
           training_instance_type,
           model_approval_status,
       ],
       steps=[processing_step, training_step, register_step],
   )
   
   # Execute pipeline
   execution = pipeline.start()
   ```

**Verification Checklist:**
- ✅ SageMaker environment set up with notebook instance
- ✅ Training data prepared and uploaded to S3
- ✅ Training script created with proper structure
- ✅ Model training job completed successfully
- ✅ Hyperparameter tuning job optimized model
- ✅ Model registered in SageMaker Model Registry
- ✅ Real-time endpoint deployed and tested
- ✅ Batch transform job for large-scale inference
- ✅ Model monitoring and drift detection enabled
- ✅ A/B testing framework implemented
- ✅ MLOps pipeline created for automation