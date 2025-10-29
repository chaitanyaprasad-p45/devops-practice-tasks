# Linux & Amazon EC2 Beginner's Guide

## Table of Contents
1. [Introduction to Linux](#introduction-to-linux)
2. [Linux Distributions](#linux-distributions)
3. [Linux File System](#linux-file-system)
4. [Essential Linux Commands](#essential-linux-commands)
5. [Text Editors](#text-editors)
6. [File Permissions and Ownership](#file-permissions-and-ownership)
7. [Process Management](#process-management)
8. [System Monitoring](#system-monitoring)
9. [Package Management](#package-management)
10. [Introduction to Amazon EC2](#introduction-to-amazon-ec2)
11. [Launching EC2 Instances](#launching-ec2-instances)
12. [Connecting to EC2 Instances](#connecting-to-ec2-instances)
13. [EC2 Security](#ec2-security)
14. [Storage and Networking](#storage-and-networking)
15. [System Administration on EC2](#system-administration-on-ec2)

## Introduction to Linux

### What is Linux?

Linux is a free and open-source operating system kernel first released by Linus Torvalds in 1991. It's the foundation for many operating systems (called distributions) and powers everything from smartphones to supercomputers.

### Why Learn Linux?

- **Server dominance**: 96.4% of top 1 million web servers run Linux
- **Cloud computing**: Most cloud instances run Linux
- **DevOps**: Essential for automation and deployment
- **Cost-effective**: Free and open-source
- **Stability**: Reliable and secure
- **Customizable**: Highly configurable

### Linux vs Windows

| Feature | Linux | Windows |
|---------|-------|---------|
| Cost | Free | Paid |
| Source Code | Open source | Proprietary |
| Security | Generally more secure | More vulnerable |
| Command Line | Powerful terminal | PowerShell (improved) |
| Customization | Highly customizable | Limited customization |
| Learning Curve | Steeper for beginners | More familiar interface |

### Linux Philosophy

- **Everything is a file**: Devices, processes, and resources are represented as files
- **Small, focused tools**: Each program does one thing well
- **Pipe and combine**: Connect simple tools to create powerful solutions
- **Configuration in text files**: Easy to edit and version control

## Linux Distributions

### Popular Distributions

**Ubuntu**
- Beginner-friendly
- Based on Debian
- Large community support
- Regular releases (LTS every 2 years)

**CentOS/RHEL/Rocky Linux**
- Enterprise-focused
- Stable and secure
- Long-term support
- Package management with YUM/DNF

**Amazon Linux**
- Optimized for AWS
- Includes AWS tools
- Based on RHEL/CentOS
- Free on AWS

**Debian**
- Very stable
- Large package repository
- Conservative release cycle
- Strong community

### Choosing a Distribution

**For Beginners**: Ubuntu, Linux Mint
**For Servers**: Ubuntu Server, CentOS, RHEL
**For AWS**: Amazon Linux 2
**For Containers**: Alpine Linux

## Linux File System

### File System Hierarchy

```
/                    (Root directory)
├── bin/            (Essential binaries)
├── boot/           (Boot loader files)
├── dev/            (Device files)
├── etc/            (Configuration files)
├── home/           (User home directories)
├── lib/            (Shared libraries)
├── media/          (Removable media)
├── mnt/            (Mount points)
├── opt/            (Optional software)
├── proc/           (Process information)
├── root/           (Root user home)
├── run/            (Runtime data)
├── sbin/           (System binaries)
├── srv/            (Service data)
├── sys/            (System information)
├── tmp/            (Temporary files)
├── usr/            (User programs)
└── var/            (Variable data)
```

### Important Directories

**`/etc`**: Configuration files
- `/etc/passwd`: User accounts
- `/etc/shadow`: Encrypted passwords
- `/etc/hosts`: Host name resolution
- `/etc/fstab`: File system mount table

**`/var`**: Variable data
- `/var/log`: Log files
- `/var/www`: Web server files
- `/var/lib`: Application data

**`/usr`**: User programs
- `/usr/bin`: User binaries
- `/usr/local`: Locally installed software

**`/home`**: User home directories
- `/home/username`: User's personal directory

### File Types

- **Regular files**: Text, binary, executable files
- **Directories**: Folders containing other files
- **Symbolic links**: Shortcuts to other files
- **Device files**: Represent hardware devices
- **Named pipes**: Inter-process communication
- **Sockets**: Network communication endpoints

## Essential Linux Commands

### Navigation Commands

```bash
# Print working directory
pwd

# List directory contents
ls
ls -l          # Long format
ls -la         # Include hidden files
ls -lh         # Human readable sizes
ls -lt         # Sort by modification time

# Change directory
cd /path/to/directory
cd ~           # Go to home directory
cd ..          # Go to parent directory
cd -           # Go to previous directory

# Create directory
mkdir directory_name
mkdir -p path/to/directory    # Create parent directories

# Remove directory
rmdir directory_name          # Remove empty directory
rm -rf directory_name         # Remove directory and contents
```

### File Operations

```bash
# Create empty file
touch filename.txt

# Copy files
cp source.txt destination.txt
cp -r source_dir/ dest_dir/   # Copy directory recursively

# Move/rename files
mv oldname.txt newname.txt
mv file.txt /path/to/directory/

# Remove files
rm filename.txt
rm -i filename.txt            # Interactive removal
rm -f filename.txt            # Force removal

# Find files
find /path -name "*.txt"
find . -type f -name "config*"
find /var/log -mtime -7       # Files modified in last 7 days

# Locate files (faster, uses database)
locate filename
updatedb                      # Update locate database
```

### File Content Commands

```bash
# Display file contents
cat filename.txt              # Display entire file
less filename.txt             # Page through file
more filename.txt             # Page through file (older)
head filename.txt             # First 10 lines
head -n 20 filename.txt       # First 20 lines
tail filename.txt             # Last 10 lines
tail -f /var/log/syslog       # Follow log file in real-time

# Search in files
grep "pattern" filename.txt
grep -r "pattern" /path/      # Recursive search
grep -i "pattern" file.txt    # Case insensitive
grep -v "pattern" file.txt    # Invert match (exclude pattern)
grep -n "pattern" file.txt    # Show line numbers

# Count lines, words, characters
wc filename.txt
wc -l filename.txt            # Count lines only
wc -w filename.txt            # Count words only
```

### File Comparison and Sorting

```bash
# Compare files
diff file1.txt file2.txt
diff -u file1.txt file2.txt   # Unified format

# Sort file contents
sort filename.txt
sort -r filename.txt          # Reverse sort
sort -n numbers.txt           # Numeric sort
sort -k 2 data.txt            # Sort by second column

# Remove duplicate lines
uniq filename.txt
sort filename.txt | uniq      # Sort then remove duplicates
sort filename.txt | uniq -c   # Count occurrences
```

### Text Processing

```bash
# Cut columns from text
cut -d',' -f1,3 data.csv      # Extract columns 1 and 3 from CSV
cut -c1-10 filename.txt       # Extract characters 1-10

# Replace text
sed 's/old/new/g' filename.txt
sed -i 's/old/new/g' file.txt # Edit file in-place

# Pattern processing
awk '{print $1}' filename.txt     # Print first column
awk -F',' '{print $1,$3}' data.csv   # Use comma as delimiter
awk '{sum+=$1} END {print sum}' numbers.txt   # Sum first column
```

### Archive and Compression

```bash
# Create tar archive
tar -cvf archive.tar file1 file2 directory/
tar -czvf archive.tar.gz files/    # Compress with gzip
tar -cjvf archive.tar.bz2 files/   # Compress with bzip2

# Extract tar archive
tar -xvf archive.tar
tar -xzvf archive.tar.gz
tar -xjvf archive.tar.bz2

# List archive contents
tar -tvf archive.tar

# Zip files
zip archive.zip file1 file2
zip -r archive.zip directory/

# Unzip files
unzip archive.zip
unzip -l archive.zip          # List contents
```

### Network Commands

```bash
# Download files
wget https://example.com/file.txt
curl -O https://example.com/file.txt
curl -L -o file.txt https://example.com/redirect

# Network information
ping google.com
ping -c 4 google.com          # Send 4 packets
traceroute google.com
nslookup google.com
dig google.com

# Network connections
netstat -tuln                 # Show listening ports
ss -tuln                      # Modern replacement for netstat
lsof -i :80                   # Show what's using port 80
```

## Text Editors

### nano (Beginner-friendly)

```bash
# Open file with nano
nano filename.txt

# Basic nano commands (shown at bottom of screen):
# Ctrl+O: Write (save) file
# Ctrl+X: Exit
# Ctrl+W: Search
# Ctrl+K: Cut line
# Ctrl+U: Paste
# Ctrl+G: Help
```

### vim (Advanced but powerful)

```bash
# Open file with vim
vim filename.txt

# vim modes:
# Normal mode: Default mode for navigation
# Insert mode: For editing text
# Command mode: For executing commands
```

**Basic vim commands:**
```bash
# Enter insert mode
i                    # Insert before cursor
a                    # Insert after cursor
o                    # Insert new line below

# Save and exit
:w                   # Save
:q                   # Quit
:wq                  # Save and quit
:q!                  # Quit without saving

# Navigation (normal mode)
h, j, k, l          # Left, down, up, right
w                   # Next word
b                   # Previous word
0                   # Beginning of line
$                   # End of line
gg                  # First line
G                   # Last line

# Search
/pattern            # Search forward
?pattern            # Search backward
n                   # Next match
N                   # Previous match

# Copy, cut, paste
yy                  # Copy line
dd                  # Cut line
p                   # Paste after cursor
P                   # Paste before cursor
```

### emacs (Alternative advanced editor)

```bash
# Open file with emacs
emacs filename.txt

# Basic emacs commands:
# Ctrl+x Ctrl+s: Save
# Ctrl+x Ctrl+c: Exit
# Ctrl+s: Search forward
# Ctrl+r: Search backward
# Ctrl+k: Cut line
# Ctrl+y: Paste
```

## File Permissions and Ownership

### Understanding Permissions

Every file and directory has three types of permissions for three categories of users:

**Permission Types:**
- **r (read)**: View file contents or list directory
- **w (write)**: Modify file or add/remove files in directory
- **x (execute)**: Run file as program or access directory

**User Categories:**
- **Owner (u)**: The user who owns the file
- **Group (g)**: Users in the file's group
- **Others (o)**: All other users

### Viewing Permissions

```bash
ls -l filename.txt
# Output: -rw-r--r-- 1 user group 1234 Oct 17 10:30 filename.txt
#         |||||||||| | |    |     |    |           |
#         |||||||||| | |    |     |    |           filename
#         |||||||||| | |    |     |    modification time
#         |||||||||| | |    |     file size
#         |||||||||| | |    group owner
#         |||||||||| | user owner
#         |||||||||| number of links
#         ||||||||||
#         |||||||||others permissions (r--)
#         ||||||group permissions (r--)
#         |||owner permissions (rw-)
#         ||file type (- = regular file, d = directory, l = link)
```

### Changing Permissions

**Symbolic method:**
```bash
# Add permissions
chmod u+x filename.txt        # Add execute for owner
chmod g+w filename.txt        # Add write for group
chmod o+r filename.txt        # Add read for others
chmod a+x filename.txt        # Add execute for all

# Remove permissions
chmod u-x filename.txt        # Remove execute for owner
chmod g-w filename.txt        # Remove write for group
chmod o-r filename.txt        # Remove read for others

# Set exact permissions
chmod u=rw,g=r,o=r filename.txt
```

**Numeric method:**
```bash
# Permission values:
# r = 4, w = 2, x = 1
# Add values for desired permissions

chmod 755 script.sh           # rwxr-xr-x (7=4+2+1, 5=4+1, 5=4+1)
chmod 644 document.txt        # rw-r--r-- (6=4+2, 4=4, 4=4)
chmod 600 private.txt         # rw------- (6=4+2, 0=0, 0=0)
chmod 777 public_script.sh    # rwxrwxrwx (full permissions)
```

**Common permission combinations:**
- **755**: Executable files (scripts, programs)
- **644**: Regular files (documents, configuration files)
- **600**: Private files (SSH keys, passwords)
- **777**: Fully accessible (rarely recommended)

### Changing Ownership

```bash
# Change owner
chown newowner filename.txt
chown newowner:newgroup filename.txt

# Change group
chgrp newgroup filename.txt

# Recursive changes
chown -R user:group directory/
chmod -R 755 directory/
```

### Special Permissions

```bash
# Set UID (SUID) - run with owner's privileges
chmod u+s executable

# Set GID (SGID) - inherit group ownership
chmod g+s directory/

# Sticky bit - only owner can delete files
chmod +t /tmp/
```

## Process Management

### Viewing Processes

```bash
# List running processes
ps                    # Current terminal processes
ps aux                # All processes, detailed
ps -ef                # All processes, different format

# Real-time process monitoring
top                   # Interactive process viewer
htop                  # Enhanced version (if installed)

# Process tree
pstree                # Show process hierarchy
ps aux --forest       # Tree view with ps
```

### Managing Processes

```bash
# Run command in background
command &

# Bring to foreground
fg

# Send to background
bg

# List jobs
jobs

# Kill processes
kill PID              # Graceful termination (SIGTERM)
kill -9 PID           # Force kill (SIGKILL)
kill -15 PID          # Explicit SIGTERM
killall process_name  # Kill by name
pkill pattern         # Kill by pattern

# Find process by name
pgrep process_name
pidof process_name
```

### Process Control Signals

```bash
# Common signals:
# SIGTERM (15): Graceful termination
# SIGKILL (9): Force termination
# SIGSTOP (19): Pause process
# SIGCONT (18): Resume process
# SIGHUP (1): Hangup (often used to reload config)

# Examples:
kill -TERM 1234       # Graceful termination
kill -KILL 1234       # Force kill
kill -STOP 1234       # Pause process
kill -CONT 1234       # Resume process
kill -HUP 1234        # Send hangup signal
```

### Running Processes Continuously

```bash
# Run command even after logout
nohup command &

# Using screen (persistent terminal sessions)
screen                # Start new screen session
screen -S name        # Start named session
screen -r             # Reattach to session
screen -list          # List sessions
# Ctrl+A, D: Detach from screen
# Ctrl+A, K: Kill screen session

# Using tmux (modern alternative to screen)
tmux                  # Start new session
tmux new -s name      # Start named session
tmux attach -t name   # Attach to session
tmux list-sessions    # List sessions
# Ctrl+B, D: Detach from tmux
# Ctrl+B, K: Kill tmux session
```

## System Monitoring

### System Information

```bash
# System information
uname -a              # System information
hostname              # Computer name
whoami                # Current username
id                    # User and group IDs
uptime                # System uptime and load
date                  # Current date and time

# Hardware information
lscpu                 # CPU information
lshw                  # Hardware information (detailed)
lsblk                 # Block devices
lsusb                 # USB devices
lspci                 # PCI devices
dmidecode             # DMI/SMBIOS information
```

### Resource Monitoring

```bash
# Memory usage
free -h               # Memory usage (human readable)
cat /proc/meminfo     # Detailed memory information

# Disk usage
df -h                 # Disk space usage
du -h directory/      # Directory size
du -sh *              # Size of all items in current directory
ncdu                  # Interactive disk usage analyzer

# CPU and load
top                   # Real-time system stats
htop                  # Enhanced top
iostat                # I/O statistics
vmstat                # Virtual memory statistics
sar                   # System activity reporter

# Network monitoring
iftop                 # Network bandwidth usage
nethogs               # Network usage by process
ss -tuln              # Network connections
netstat -i            # Network interface statistics
```

### Log Files

```bash
# System logs
tail -f /var/log/syslog       # Follow system log
tail -f /var/log/messages     # System messages (RHEL/CentOS)
tail -f /var/log/secure       # Authentication log (RHEL/CentOS)
tail -f /var/log/auth.log     # Authentication log (Ubuntu/Debian)

# Application logs
tail -f /var/log/apache2/access.log   # Apache access log
tail -f /var/log/nginx/error.log      # Nginx error log

# Using journalctl (systemd systems)
journalctl            # View all logs
journalctl -f         # Follow logs
journalctl -u service # Logs for specific service
journalctl --since "1 hour ago"  # Recent logs
journalctl -p err     # Error level logs only
```

## Package Management

### Ubuntu/Debian (APT)

```bash
# Update package list
sudo apt update

# Upgrade packages
sudo apt upgrade
sudo apt full-upgrade         # More comprehensive upgrade

# Install packages
sudo apt install package_name
sudo apt install package1 package2

# Remove packages
sudo apt remove package_name
sudo apt purge package_name   # Remove including config files
sudo apt autoremove           # Remove unused dependencies

# Search packages
apt search keyword
apt list --installed         # List installed packages
apt list --upgradable        # List upgradable packages

# Package information
apt show package_name
apt policy package_name      # Show available versions

# Fix broken packages
sudo apt --fix-broken install
sudo dpkg --configure -a
```

### CentOS/RHEL/Rocky Linux (YUM/DNF)

```bash
# Update package list and upgrade
sudo yum update              # CentOS 7 and earlier
sudo dnf update              # CentOS 8+, Fedora

# Install packages
sudo yum install package_name
sudo dnf install package_name

# Remove packages
sudo yum remove package_name
sudo dnf remove package_name

# Search packages
yum search keyword
dnf search keyword

# List packages
yum list installed
dnf list installed
yum list available
dnf list available

# Package information
yum info package_name
dnf info package_name

# Clean cache
yum clean all
dnf clean all
```

### Amazon Linux (YUM)

```bash
# Update system
sudo yum update

# Install packages
sudo yum install package_name

# Install from Amazon Linux Extras
sudo amazon-linux-extras install topic

# List available extras
amazon-linux-extras list

# Common installations
sudo yum install git
sudo yum install docker
sudo amazon-linux-extras install docker
sudo amazon-linux-extras install nginx1
```

### Installing from Source

```bash
# Download source code
wget https://example.com/software-1.0.tar.gz
tar -xzf software-1.0.tar.gz
cd software-1.0/

# Typical build process
./configure           # Configure build options
make                  # Compile software
sudo make install     # Install software

# Alternative with cmake
mkdir build && cd build
cmake ..
make
sudo make install
```

## Introduction to Amazon EC2

### What is Amazon EC2?

Amazon Elastic Compute Cloud (EC2) is a web service that provides resizable compute capacity in the cloud. Think of it as virtual servers that you can rent and configure according to your needs.

### Key Benefits of EC2

- **Scalability**: Scale capacity up or down within minutes
- **Flexibility**: Choose from various instance types and operating systems
- **Cost-effective**: Pay only for what you use
- **Reliable**: 99.99% SLA with multiple availability zones
- **Secure**: Integrated with AWS security services
- **Global**: Available in multiple regions worldwide

### EC2 Instance Types

**General Purpose:**
- **t3, t4g**: Burstable performance instances
- **m5, m6i**: Balanced compute, memory, and networking

**Compute Optimized:**
- **c5, c6i**: High-performance processors

**Memory Optimized:**
- **r5, r6i**: Fast performance for memory-intensive applications
- **x1e, x2**: High memory-to-vCPU ratio

**Storage Optimized:**
- **i3, i4i**: High sequential read and write access
- **d2, d3**: Dense HDD storage

**Accelerated Computing:**
- **p3, p4**: GPU instances for machine learning
- **g4**: GPU instances for graphics workloads

### Instance Sizes

```
nano    1 vCPU    0.5 GB RAM
micro   1 vCPU    1 GB RAM
small   1 vCPU    2 GB RAM
medium  1 vCPU    4 GB RAM
large   2 vCPU    8 GB RAM
xlarge  4 vCPU    16 GB RAM
2xlarge 8 vCPU    32 GB RAM
... and larger sizes available
```

## Launching EC2 Instances

### Using AWS Management Console

**Step 1: Choose AMI (Amazon Machine Image)**
- Amazon Linux 2
- Ubuntu Server 20.04 LTS
- Red Hat Enterprise Linux
- Windows Server
- Custom AMIs

**Step 2: Choose Instance Type**
- Consider your workload requirements
- Start with t3.micro for testing (free tier eligible)

**Step 3: Configure Instance Details**
- Number of instances
- VPC and subnet
- IAM role
- User data (startup scripts)

**Step 4: Add Storage**
- Root volume (default: 8GB)
- Additional EBS volumes
- Choose volume type (gp3, gp2, io1, io2)

**Step 5: Add Tags**
- Key-value pairs for organization
- Example: Name=WebServer, Environment=Production

**Step 6: Configure Security Group**
- Virtual firewall rules
- Specify allowed ports and source IPs

**Step 7: Review and Launch**
- Select or create key pair for SSH access

### Using AWS CLI

```bash
# Launch instance
aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --count 1 \
    --instance-type t3.micro \
    --key-name my-key-pair \
    --security-group-ids sg-12345678 \
    --subnet-id subnet-12345678 \
    --user-data file://user-data.sh

# List instances
aws ec2 describe-instances

# Get instance IDs
aws ec2 describe-instances \
    --query 'Reservations[*].Instances[*].InstanceId' \
    --output text

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

### User Data Scripts

User data allows you to pass scripts to your instance at launch time:

```bash
#!/bin/bash
# Update system
yum update -y

# Install Apache web server
yum install -y httpd

# Start and enable Apache
systemctl start httpd
systemctl enable httpd

# Create a simple web page
echo "<h1>Hello from EC2!</h1>" > /var/www/html/index.html

# Install and configure CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm
```

## Connecting to EC2 Instances

### SSH Connection

**Prerequisites:**
- Key pair (.pem file)
- Security group allowing SSH (port 22)
- Public IP or Elastic IP

**Connect from Linux/Mac:**
```bash
# Set correct permissions for key file
chmod 400 my-key.pem

# Connect to instance
ssh -i my-key.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com

# Connect with verbose output for troubleshooting
ssh -v -i my-key.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com

# Copy files to instance
scp -i my-key.pem file.txt ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:~/

# Copy files from instance
scp -i my-key.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:~/file.txt ./
```

**Connect from Windows:**

Using PuTTY:
1. Convert .pem to .ppk using PuTTYgen
2. Open PuTTY
3. Enter hostname: ec2-user@public-ip
4. SSH > Auth > Browse to .ppk file
5. Open connection

Using Windows 10/11 OpenSSH:
```powershell
# Same as Linux/Mac
ssh -i my-key.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```

### Default Usernames by AMI

- **Amazon Linux**: ec2-user
- **Ubuntu**: ubuntu
- **CentOS**: centos or ec2-user
- **RHEL**: ec2-user or root
- **Debian**: admin
- **SUSE**: ec2-user

### EC2 Instance Connect

Browser-based SSH connection:
1. Go to EC2 console
2. Select instance
3. Click "Connect"
4. Choose "EC2 Instance Connect"
5. Click "Connect"

### Session Manager

Secure shell access without SSH:
1. Attach IAM role with SSM permissions to instance
2. Install SSM agent (pre-installed on newer AMIs)
3. Use Session Manager in EC2 console

```bash
# Install Session Manager plugin for AWS CLI
# Then connect using:
aws ssm start-session --target i-1234567890abcdef0
```

## EC2 Security

### Security Groups

Security groups act as virtual firewalls for your instances:

```bash
# Create security group
aws ec2 create-security-group \
    --group-name web-server-sg \
    --description "Security group for web server"

# Add inbound rules
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 22 \
    --cidr 203.0.113.0/24

# List security groups
aws ec2 describe-security-groups

# Remove rules
aws ec2 revoke-security-group-ingress \
    --group-id sg-12345678 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

**Common Security Group Rules:**
- SSH (22): Your IP address only
- HTTP (80): 0.0.0.0/0 (public web servers)
- HTTPS (443): 0.0.0.0/0 (public web servers)
- MySQL (3306): Application server security group
- PostgreSQL (5432): Application server security group

### Key Pairs

```bash
# Create key pair
aws ec2 create-key-pair \
    --key-name my-key \
    --query 'KeyMaterial' \
    --output text > my-key.pem

# Import existing public key
aws ec2 import-key-pair \
    --key-name my-imported-key \
    --public-key-material fileb://~/.ssh/id_rsa.pub

# List key pairs
aws ec2 describe-key-pairs

# Delete key pair
aws ec2 delete-key-pair --key-name my-key
```

### IAM Roles for EC2

```bash
# Create role trust policy
cat > trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create IAM role
aws iam create-role \
    --role-name EC2-S3-Access \
    --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
    --role-name EC2-S3-Access \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile --instance-profile-name EC2-S3-Access

# Add role to instance profile
aws iam add-role-to-instance-profile \
    --instance-profile-name EC2-S3-Access \
    --role-name EC2-S3-Access

# Attach instance profile to EC2
aws ec2 associate-iam-instance-profile \
    --instance-id i-1234567890abcdef0 \
    --iam-instance-profile Name=EC2-S3-Access
```

### Security Best Practices

1. **Principle of least privilege**: Only allow necessary access
2. **Regular updates**: Keep OS and software updated
3. **Strong passwords**: Use complex passwords for additional users
4. **Disable root login**: Use sudo instead
5. **Enable logging**: Monitor access and activities
6. **Use private subnets**: For non-public facing instances
7. **Regular backups**: Backup important data
8. **Monitor unusual activity**: Use CloudTrail and CloudWatch

## Storage and Networking

### EBS (Elastic Block Store)

**Volume Types:**
- **gp3**: General purpose SSD (newer, more flexible)
- **gp2**: General purpose SSD (older)
- **io2**: Provisioned IOPS SSD (high performance)
- **st1**: Throughput optimized HDD
- **sc1**: Cold HDD (lowest cost)

```bash
# Create EBS volume
aws ec2 create-volume \
    --size 100 \
    --volume-type gp3 \
    --availability-zone us-west-2a

# Attach volume to instance
aws ec2 attach-volume \
    --volume-id vol-12345678 \
    --instance-id i-1234567890abcdef0 \
    --device /dev/sdf

# List volumes
aws ec2 describe-volumes

# Create snapshot
aws ec2 create-snapshot \
    --volume-id vol-12345678 \
    --description "Backup of web server data"

# Create volume from snapshot
aws ec2 create-volume \
    --snapshot-id snap-12345678 \
    --availability-zone us-west-2a
```

**Mount EBS Volume on Instance:**
```bash
# Check available disks
lsblk

# Create file system (first time only)
sudo mkfs -t ext4 /dev/xvdf

# Create mount point
sudo mkdir /data

# Mount volume
sudo mount /dev/xvdf /data

# Add to fstab for permanent mounting
echo '/dev/xvdf /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab

# Verify mount
df -h
```

### Instance Store

```bash
# Instance store volumes are already attached
# Check available disks
lsblk

# Format and mount (ephemeral storage)
sudo mkfs -t ext4 /dev/nvme1n1
sudo mkdir /tmp-storage
sudo mount /dev/nvme1n1 /tmp-storage
```

### Elastic IP

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with instance
aws ec2 associate-address \
    --instance-id i-1234567890abcdef0 \
    --allocation-id eipalloc-12345678

# List Elastic IPs
aws ec2 describe-addresses

# Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-12345678
```

### VPC and Networking

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnet
aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-west-2a

# Create internet gateway
aws ec2 create-internet-gateway

# Attach internet gateway to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id igw-12345678 \
    --vpc-id vpc-12345678

# Create route table
aws ec2 create-route-table --vpc-id vpc-12345678

# Add route to internet
aws ec2 create-route \
    --route-table-id rtb-12345678 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-12345678

# Associate subnet with route table
aws ec2 associate-route-table \
    --subnet-id subnet-12345678 \
    --route-table-id rtb-12345678
```

## System Administration on EC2

### User Management

```bash
# Create new user
sudo useradd -m -s /bin/bash newuser

# Set password
sudo passwd newuser

# Add user to sudo group
sudo usermod -aG wheel newuser     # CentOS/RHEL
sudo usermod -aG sudo newuser      # Ubuntu/Debian

# Create SSH directory for user
sudo mkdir /home/newuser/.ssh
sudo chmod 700 /home/newuser/.ssh

# Add public key for SSH access
sudo cp ~/.ssh/authorized_keys /home/newuser/.ssh/
sudo chown -R newuser:newuser /home/newuser/.ssh
sudo chmod 600 /home/newuser/.ssh/authorized_keys

# Switch to user
sudo su - newuser

# List users
cat /etc/passwd
cut -d: -f1 /etc/passwd

# Delete user
sudo userdel -r username
```

### Service Management (systemd)

```bash
# Start service
sudo systemctl start httpd

# Stop service
sudo systemctl stop httpd

# Restart service
sudo systemctl restart httpd

# Reload configuration
sudo systemctl reload httpd

# Enable service (start at boot)
sudo systemctl enable httpd

# Disable service
sudo systemctl disable httpd

# Check service status
sudo systemctl status httpd

# List all services
sudo systemctl list-units --type=service

# List enabled services
sudo systemctl list-unit-files --type=service --state=enabled

# View service logs
sudo journalctl -u httpd
sudo journalctl -u httpd -f    # Follow logs
```

### Cron Jobs

```bash
# Edit crontab for current user
crontab -e

# Edit crontab for specific user
sudo crontab -u username -e

# List cron jobs
crontab -l

# Cron format:
# * * * * * command
# | | | | |
# | | | | day of week (0-7, Sunday = 0 or 7)
# | | | month (1-12)
# | | day of month (1-31)
# | hour (0-23)
# minute (0-59)

# Examples:
# Run every minute
* * * * * /path/to/script.sh

# Run daily at 2:30 AM
30 2 * * * /path/to/backup.sh

# Run every Monday at 9:00 AM
0 9 * * 1 /path/to/weekly-task.sh

# Run on the 1st of every month at midnight
0 0 1 * * /path/to/monthly-task.sh

# System cron jobs
sudo nano /etc/crontab
ls /etc/cron.d/
ls /etc/cron.daily/
ls /etc/cron.weekly/
ls /etc/cron.monthly/
```

### Log Management

```bash
# Configure log rotation
sudo nano /etc/logrotate.conf

# Custom log rotation
sudo nano /etc/logrotate.d/myapp

# Example logrotate configuration:
/var/log/myapp/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 myapp myapp
    postrotate
        systemctl reload myapp
    endscript
}

# Test log rotation
sudo logrotate -d /etc/logrotate.conf
sudo logrotate -f /etc/logrotate.d/myapp
```

### Firewall Configuration

**UFW (Ubuntu):**
```bash
# Enable firewall
sudo ufw enable

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow specific ports
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443

# Allow from specific IP
sudo ufw allow from 203.0.113.0/24

# Allow specific service
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# List rules
sudo ufw status
sudo ufw status numbered

# Delete rules
sudo ufw delete 2
sudo ufw delete allow 80

# Disable firewall
sudo ufw disable
```

**firewalld (CentOS/RHEL):**
```bash
# Start and enable firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Check status
sudo firewall-cmd --state

# List zones
sudo firewall-cmd --get-zones
sudo firewall-cmd --get-default-zone

# List services in zone
sudo firewall-cmd --list-services
sudo firewall-cmd --zone=public --list-services

# Add service
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent

# Add port
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent

# Reload configuration
sudo firewall-cmd --reload

# Remove service
sudo firewall-cmd --zone=public --remove-service=http --permanent
```

### Performance Tuning

```bash
# Check system limits
ulimit -a

# Increase file descriptor limit
echo "* soft nofile 65536" | sudo tee -a /etc/security/limits.conf
echo "* hard nofile 65536" | sudo tee -a /etc/security/limits.conf

# Kernel parameters
sudo nano /etc/sysctl.conf

# Common kernel tuning parameters:
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 65536 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5

# Apply changes
sudo sysctl -p

# Monitor performance
iostat -x 1
vmstat 1
sar -u 1 10
```

### Backup Strategies

```bash
# Create EBS snapshot (backup)
aws ec2 create-snapshot \
    --volume-id vol-12345678 \
    --description "Daily backup $(date +%Y-%m-%d)"

# Automated backup script
#!/bin/bash
# backup.sh

VOLUME_ID="vol-12345678"
DESCRIPTION="Automated backup $(date +%Y-%m-%d_%H-%M-%S)"

# Create snapshot
SNAPSHOT_ID=$(aws ec2 create-snapshot \
    --volume-id $VOLUME_ID \
    --description "$DESCRIPTION" \
    --query 'SnapshotId' \
    --output text)

echo "Created snapshot: $SNAPSHOT_ID"

# Delete snapshots older than 7 days
aws ec2 describe-snapshots \
    --owner-ids self \
    --filters "Name=volume-id,Values=$VOLUME_ID" \
    --query 'Snapshots[?StartTime<=`'"$(date -d '7 days ago' -u +%Y-%m-%dT%H:%M:%S.000Z)"'`].SnapshotId' \
    --output text | xargs -n 1 aws ec2 delete-snapshot --snapshot-id

# File system backup with rsync
rsync -avz --delete /var/www/ /backup/www/
rsync -avz --delete /etc/ /backup/etc/

# Database backup
mysqldump -u root -p database_name > /backup/database_$(date +%Y%m%d).sql
pg_dump database_name > /backup/database_$(date +%Y%m%d).sql
```

### Auto Scaling and Load Balancing

```bash
# Create launch template
aws ec2 create-launch-template \
    --launch-template-name web-server-template \
    --launch-template-data '{
        "ImageId": "ami-0abcdef1234567890",
        "InstanceType": "t3.micro",
        "KeyName": "my-key",
        "SecurityGroupIds": ["sg-12345678"],
        "UserData": "'$(base64 -w 0 user-data.sh)'"
    }'

# Create Auto Scaling group
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name web-server-asg \
    --launch-template LaunchTemplateName=web-server-template,Version=1 \
    --min-size 1 \
    --max-size 5 \
    --desired-capacity 2 \
    --vpc-zone-identifier "subnet-12345678,subnet-87654321"

# Create load balancer
aws elbv2 create-load-balancer \
    --name web-server-lb \
    --subnets subnet-12345678 subnet-87654321 \
    --security-groups sg-12345678

# Create target group
aws elbv2 create-target-group \
    --name web-server-targets \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-12345678 \
    --health-check-path /health
```

## Next Steps

### Learning Path

1. **Master Linux Basics**
   - Practice command line daily
   - Set up local Linux environment
   - Learn shell scripting

2. **Get Hands-on with AWS**
   - Create free tier account
   - Launch and manage EC2 instances
   - Practice with different instance types

3. **System Administration**
   - Set up web servers (Apache, Nginx)
   - Configure databases (MySQL, PostgreSQL)
   - Implement monitoring and logging

4. **Automation and DevOps**
   - Learn configuration management (Ansible)
   - Practice Infrastructure as Code (Terraform)
   - Set up CI/CD pipelines

5. **Advanced Topics**
   - Container technologies (Docker, Kubernetes)
   - Security hardening
   - Performance optimization
   - High availability architectures

### Useful Resources

**Linux Learning:**
- Linux Academy / A Cloud Guru
- LinuxCommand.org
- The Linux Documentation Project
- Ubuntu tutorials and documentation

**AWS Resources:**
- AWS Free Tier
- AWS Documentation
- AWS Training and Certification
- AWS Well-Architected Framework

**Practice Labs:**
- Set up LAMP/LEMP stack
- Configure load balancer with multiple instances
- Implement automated backups
- Set up monitoring with CloudWatch
- Create Auto Scaling group

### Certifications to Consider

- **Linux Professional Institute (LPI)**
- **Red Hat Certified System Administrator (RHCSA)**
- **AWS Certified Solutions Architect**
- **AWS Certified SysOps Administrator**

Remember: The best way to learn Linux and EC2 is through hands-on practice. Start with simple tasks and gradually build complexity as you become more comfortable with the tools and concepts!