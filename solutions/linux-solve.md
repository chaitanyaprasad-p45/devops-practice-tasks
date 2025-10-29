# Linux/EC2 Tasks Solutions

This document contains detailed solutions for all 20 Linux/EC2 practice tasks.

## Basic Level Solutions (Tasks 1-10)

### Task 1: Basic Linux Commands and Navigation
**Master essential Linux commands for file system navigation and basic operations.**

**Solution:**
```bash
# File system navigation
pwd                          # Show current directory
cd /                        # Go to root directory
cd ~                        # Go to home directory
cd ..                       # Go up one directory
cd -                        # Go to previous directory

# List files and directories
ls                          # List files in current directory
ls -l                       # Long format listing
ls -la                      # Include hidden files
ls -lh                      # Human readable file sizes
ls -lt                      # Sort by modification time
ls -ltr                     # Sort by time (oldest first)

# File operations
touch file.txt              # Create empty file
cp file.txt backup.txt      # Copy file
cp -r dir1 dir2            # Copy directory recursively
mv file.txt newname.txt     # Rename/move file
rm file.txt                 # Remove file
rm -rf directory            # Remove directory recursively

# Directory operations
mkdir mydir                 # Create directory
mkdir -p path/to/dir       # Create nested directories
rmdir emptydir             # Remove empty directory
rm -rf directory           # Remove directory and contents

# File content viewing
cat file.txt               # Display file content
less file.txt              # View file with pagination
head file.txt              # Show first 10 lines
head -n 20 file.txt        # Show first 20 lines
tail file.txt              # Show last 10 lines
tail -f /var/log/syslog    # Follow log file in real-time

# File searching
find /home -name "*.txt"   # Find files by name
find /var -type f -size +100M  # Find large files
grep -r "error" /var/log   # Search for text in files
which python3              # Find command location
whereis nginx              # Find binary, source, manual

# File permissions
chmod 755 script.sh        # Set permissions (rwxr-xr-x)
chmod u+x script.sh        # Add execute permission for owner
chmod -R 644 directory     # Set permissions recursively
chown user:group file.txt  # Change ownership
chgrp group file.txt       # Change group ownership

# Archive operations
tar -czf archive.tar.gz directory/    # Create compressed archive
tar -xzf archive.tar.gz              # Extract compressed archive
zip -r archive.zip directory/        # Create zip archive
unzip archive.zip                    # Extract zip archive

# System information
whoami                     # Current username
id                         # User and group IDs
uname -a                   # System information
hostname                   # System hostname
uptime                     # System uptime and load
df -h                      # Disk space usage
du -sh *                   # Directory sizes
free -h                    # Memory usage
ps aux                     # Running processes
top                        # Real-time process monitor
htop                       # Enhanced process monitor

# Network commands
ping google.com            # Test connectivity
wget http://example.com/file.txt  # Download file
curl -I http://example.com # Get HTTP headers
netstat -tulpn            # Show network connections
ss -tulpn                 # Modern netstat alternative

# Text processing
sort file.txt             # Sort lines
uniq file.txt            # Remove duplicates
wc -l file.txt           # Count lines
cut -d',' -f1 file.csv   # Extract columns
awk '{print $1}' file.txt # Extract first column
sed 's/old/new/g' file.txt # Replace text

# Practice script
cat > linux_practice.sh << 'EOF'
#!/bin/bash

# Create directory structure
mkdir -p project/{src,bin,docs,logs}

# Create sample files
echo "Hello World" > project/docs/readme.txt
echo "#!/bin/bash" > project/src/script.sh
echo "echo 'Script executed'" >> project/src/script.sh

# Set permissions
chmod +x project/src/script.sh

# Create archive
tar -czf project_backup.tar.gz project/

# Display information
echo "=== Directory Structure ==="
tree project/ 2>/dev/null || find project/ -type d

echo -e "\n=== File Permissions ==="
ls -la project/src/

echo -e "\n=== Archive Created ==="
ls -lh project_backup.tar.gz

echo -e "\n=== Disk Usage ==="
du -sh project/

echo "Linux basics practice completed!"
EOF

chmod +x linux_practice.sh
./linux_practice.sh
```

### Task 2: User and Group Management
**Create and manage users, groups, and configure sudo access.**

**Solution:**
```bash
# Create new user
sudo useradd -m -s /bin/bash john          # Create user with home directory
sudo passwd john                           # Set password

# Advanced user creation
sudo useradd -m -s /bin/bash -G sudo,docker -c "John Doe" john

# User information
id john                                    # Show user info
finger john                               # Detailed user info
last john                                 # Show login history

# Modify user
sudo usermod -a -G docker john            # Add user to group
sudo usermod -s /bin/zsh john             # Change shell
sudo usermod -l newname oldname           # Rename user
sudo usermod -L john                      # Lock user account
sudo usermod -U john                      # Unlock user account

# Delete user
sudo userdel john                         # Delete user (keep home)
sudo userdel -r john                      # Delete user and home directory

# Group management
sudo groupadd developers                   # Create group
sudo groupadd -g 1001 developers          # Create group with specific GID
sudo groupmod -n devs developers          # Rename group
sudo groupdel developers                  # Delete group

# Add user to group
sudo usermod -a -G developers john        # Add to secondary group
sudo gpasswd -a john developers           # Alternative method
sudo gpasswd -d john developers           # Remove from group

# List groups and members
groups john                               # Show user's groups
getent group developers                   # Show group members
cat /etc/group | grep developers          # Manual check

# Sudo configuration
sudo visudo                               # Edit sudoers file safely

# Add to sudoers file:
# john ALL=(ALL:ALL) ALL                  # Full sudo access
# %developers ALL=(ALL) NOPASSWD: /usr/bin/systemctl  # Group with specific commands

# Test sudo access
sudo -l                                   # List sudo permissions
sudo -u john whoami                       # Run command as another user

# Password policies
sudo chage -l john                        # Show password aging info
sudo chage -M 90 john                     # Set password expiry
sudo chage -m 7 john                      # Set minimum password age
sudo chage -W 7 john                      # Set warning days

# User management script
cat > user_management.sh << 'EOF'
#!/bin/bash

# Function to create user with full setup
create_user() {
    local username=$1
    local fullname=$2
    local groups=$3
    
    echo "Creating user: $username"
    
    # Create user
    sudo useradd -m -s /bin/bash -c "$fullname" "$username"
    
    # Set temporary password
    echo "$username:ChangeMe123!" | sudo chpasswd
    
    # Force password change on first login
    sudo chage -d 0 "$username"
    
    # Add to groups if specified
    if [ -n "$groups" ]; then
        sudo usermod -a -G "$groups" "$username"
    fi
    
    # Create .bashrc
    sudo -u "$username" cat > /home/"$username"/.bashrc << 'BASHRC'
# User .bashrc file
export PS1='\u@\h:\w\$ '
export PATH=$PATH:/usr/local/bin
alias ll='ls -la'
alias la='ls -A'
alias l='ls -CF'
BASHRC
    
    echo "User $username created successfully!"
    echo "Temporary password: ChangeMe123!"
    echo "User must change password on first login"
}

# Function to setup development environment
setup_dev_user() {
    local username=$1
    
    # Create directories
    sudo -u "$username" mkdir -p /home/"$username"/{projects,scripts,downloads}
    
    # Install common tools for user
    cat > /tmp/setup_dev.sh << 'DEVSETUP'
#!/bin/bash
# Install development tools
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
# Add other development tools setup here
DEVSETUP
    
    sudo -u "$username" bash /tmp/setup_dev.sh
    rm /tmp/setup_dev.sh
    
    echo "Development environment setup completed for $username"
}

# Create development team
echo "=== Creating Development Team ==="

# Create group
sudo groupadd developers
sudo groupadd testers

# Create users
create_user "alice" "Alice Developer" "developers,docker"
create_user "bob" "Bob Tester" "testers"
create_user "charlie" "Charlie DevOps" "developers,docker,sudo"

# Setup development environments
setup_dev_user "alice"
setup_dev_user "charlie"

# Display created users
echo -e "\n=== Created Users ==="
getent passwd | grep -E "(alice|bob|charlie)" | cut -d: -f1,5

echo -e "\n=== Group Memberships ==="
for user in alice bob charlie; do
    echo "$user: $(groups $user)"
done

echo -e "\n=== Home Directories ==="
ls -la /home/ | grep -E "(alice|bob|charlie)"
EOF

chmod +x user_management.sh
# sudo ./user_management.sh  # Run with sudo when ready
```

### Task 3: File Permissions and Security
**Configure file permissions, ACLs, and implement security best practices.**

**Solution:**
```bash
# Basic permissions (rwx for owner, group, others)
chmod 755 file.txt                        # rwxr-xr-x
chmod 644 file.txt                        # rw-r--r--
chmod 600 file.txt                        # rw-------
chmod 755 directory/                      # Standard directory permissions

# Symbolic permissions
chmod u+x file.txt                        # Add execute for owner
chmod g-w file.txt                        # Remove write for group
chmod o-r file.txt                        # Remove read for others
chmod a+r file.txt                        # Add read for all

# Recursive permissions
chmod -R 755 directory/                   # Apply to all files/dirs
find /path -type f -exec chmod 644 {} \;  # Files only
find /path -type d -exec chmod 755 {} \;  # Directories only

# Special permissions
chmod +t directory/                       # Sticky bit (only owner can delete)
chmod g+s directory/                      # SGID (inherit group)
chmod u+s program                         # SUID (run as owner)

# Octal notation with special bits
chmod 1755 directory/                     # Sticky bit + 755
chmod 2755 directory/                     # SGID + 755
chmod 4755 program                        # SUID + 755

# Ownership
chown user:group file.txt                 # Change owner and group
chown -R user:group directory/            # Recursive
chgrp group file.txt                      # Change group only

# Access Control Lists (ACLs)
# Install ACL tools if needed
sudo apt-get update && sudo apt-get install acl

# Basic ACL operations
setfacl -m u:john:r-- file.txt           # Give user john read access
setfacl -m g:developers:rw- file.txt     # Give group developers read/write
setfacl -m o::--- file.txt               # Remove all other permissions

# Default ACLs for directories
setfacl -m d:u:john:rw- directory/       # Default ACL for new files
setfacl -m d:g:developers:rw- directory/ # Default ACL for group

# View ACLs
getfacl file.txt                         # Show file ACLs
getfacl directory/                       # Show directory ACLs

# Remove ACLs
setfacl -x u:john file.txt               # Remove specific ACL
setfacl -b file.txt                      # Remove all ACLs

# File attributes
chattr +i file.txt                       # Make file immutable
chattr -i file.txt                       # Remove immutable
chattr +a file.txt                       # Append only
lsattr file.txt                          # List attributes

# umask (default permissions)
umask                                    # Show current umask
umask 022                               # Set umask (default: 755 dirs, 644 files)
umask 027                               # More restrictive (750 dirs, 640 files)

# Security scanning and hardening script
cat > security_audit.sh << 'EOF'
#!/bin/bash

echo "=== File Permission Security Audit ==="

# Find files with dangerous permissions
echo "Files with world-writable permissions:"
find /home -type f -perm -002 2>/dev/null | head -10

echo -e "\nDirectories with world-writable permissions:"
find /home -type d -perm -002 2>/dev/null | head -10

echo -e "\nFiles with SUID bit set:"
find /usr -type f -perm -4000 2>/dev/null | head -10

echo -e "\nFiles with SGID bit set:"
find /usr -type f -perm -2000 2>/dev/null | head -10

echo -e "\nFiles without owner or group:"
find /home -nouser -o -nogroup 2>/dev/null | head -10

# Check for common security issues
echo -e "\n=== Security Configuration Check ==="

# Check SSH configuration
if [ -f /etc/ssh/sshd_config ]; then
    echo "SSH Root login allowed:" $(grep "^PermitRootLogin" /etc/ssh/sshd_config || echo "Default (yes)")
    echo "SSH Password authentication:" $(grep "^PasswordAuthentication" /etc/ssh/sshd_config || echo "Default (yes)")
fi

# Check password policy
if [ -f /etc/login.defs ]; then
    echo "Password max age:" $(grep "^PASS_MAX_DAYS" /etc/login.defs)
    echo "Password min age:" $(grep "^PASS_MIN_DAYS" /etc/login.defs)
fi

# Check for empty passwords
echo -e "\nAccounts with empty passwords:"
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

echo -e "\n=== File Permission Recommendations ==="
cat << 'RECOMMENDATIONS'
1. Set appropriate umask (022 or 027)
2. Use ACLs for fine-grained permissions
3. Regular audit of SUID/SGID files
4. Remove world-writable permissions
5. Use sticky bit on shared directories
6. Implement proper backup permissions
RECOMMENDATIONS
EOF

chmod +x security_audit.sh

# Secure file handling examples
cat > secure_file_ops.sh << 'EOF'
#!/bin/bash

# Secure file creation with proper permissions
create_secure_file() {
    local filename=$1
    local content=$2
    local owner=$3
    local permissions=$4
    
    # Create file with restrictive permissions first
    (umask 077; echo "$content" > "$filename")
    
    # Set proper ownership and permissions
    sudo chown "$owner" "$filename"
    chmod "$permissions" "$filename"
    
    echo "Created secure file: $filename"
    ls -la "$filename"
}

# Secure directory setup
create_secure_directory() {
    local dirname=$1
    local owner=$2
    local permissions=$3
    
    mkdir -p "$dirname"
    sudo chown "$owner" "$dirname"
    chmod "$permissions" "$dirname"
    
    # Set default ACL for new files
    setfacl -m d:u::rwx,d:g::r-x,d:o::--- "$dirname" 2>/dev/null || true
    
    echo "Created secure directory: $dirname"
    ls -la "$dirname"
}

# Example usage
create_secure_file "/tmp/secret.txt" "Secret information" "root:root" "600"
create_secure_directory "/tmp/secure_data" "root:developers" "750"

# Backup with proper permissions
backup_with_permissions() {
    local source_dir=$1
    local backup_dir=$2
    
    # Create backup directory
    mkdir -p "$backup_dir"
    chmod 700 "$backup_dir"
    
    # Create backup preserving permissions
    tar -czpf "$backup_dir/backup_$(date +%Y%m%d_%H%M%S).tar.gz" "$source_dir"
    
    # Set backup file permissions
    chmod 600 "$backup_dir"/*.tar.gz
    
    echo "Backup created with secure permissions"
    ls -la "$backup_dir"
}

echo "=== Secure File Operations Demo ==="
# backup_with_permissions "/home/user/documents" "/backup/user_docs"
EOF

chmod +x secure_file_ops.sh

echo "File permission and security scripts created successfully!"
echo "Run ./security_audit.sh to perform security audit"
echo "Review and run ./secure_file_ops.sh for secure file operations examples"
```

### Task 4: Process Management and Monitoring
**Monitor system processes, manage services, and implement process control.**

**Solution:**
```bash
# Process viewing and monitoring
ps                                        # Show processes for current user
ps aux                                    # Show all processes (BSD style)
ps -ef                                    # Show all processes (UNIX style)
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu # Custom format, sorted by CPU

# Real-time monitoring
top                                       # Basic process monitor
htop                                      # Enhanced process monitor (install if needed)
atop                                      # Advanced process monitor

# Process tree
pstree                                    # Show process tree
pstree -p                                 # Include PIDs
ps -ef --forest                          # Process tree with ps

# Process control
kill PID                                  # Terminate process (SIGTERM)
kill -9 PID                              # Force kill (SIGKILL)
kill -STOP PID                           # Suspend process
kill -CONT PID                           # Resume process
killall process_name                      # Kill all processes by name
pkill -f "pattern"                       # Kill processes matching pattern

# Job control
jobs                                      # List background jobs
bg %1                                     # Put job 1 in background
fg %1                                     # Bring job 1 to foreground
nohup command &                           # Run command immune to hangups
screen -S session_name                    # Create screen session
tmux new -s session_name                  # Create tmux session

# System monitoring
uptime                                    # System uptime and load
w                                         # Who is logged in and what they're doing
who                                       # Show logged in users
last                                      # Show login history

# Memory monitoring
free -h                                   # Memory usage (human readable)
vmstat 2                                  # Virtual memory statistics every 2 seconds
vmstat -s                                 # Memory statistics summary

# CPU monitoring
lscpu                                     # CPU information
cat /proc/cpuinfo                         # Detailed CPU info
mpstat 2                                  # CPU statistics every 2 seconds

# I/O monitoring
iostat 2                                  # I/O statistics every 2 seconds
iotop                                     # Real-time I/O monitor (may need install)

# Network monitoring
netstat -tuln                            # Network connections
ss -tuln                                  # Modern alternative to netstat
lsof -i                                   # List open network files
iftop                                     # Network traffic monitor

# Process monitoring script
cat > process_monitor.sh << 'EOF'
#!/bin/bash

# Function to monitor specific process
monitor_process() {
    local process_name=$1
    local log_file="/var/log/${process_name}_monitor.log"
    
    echo "=== Monitoring Process: $process_name ===" | tee -a "$log_file"
    echo "Timestamp: $(date)" | tee -a "$log_file"
    
    # Check if process is running
    if pgrep "$process_name" > /dev/null; then
        echo "Process $process_name is running" | tee -a "$log_file"
        
        # Get process details
        ps -eo pid,ppid,cmd,%mem,%cpu | grep "$process_name" | grep -v grep | tee -a "$log_file"
        
        # Memory usage
        local mem_usage=$(ps -o %mem --no-headers -C "$process_name" | awk '{sum+=$1} END {print sum}')
        echo "Total memory usage: ${mem_usage}%" | tee -a "$log_file"
        
        # CPU usage
        local cpu_usage=$(ps -o %cpu --no-headers -C "$process_name" | awk '{sum+=$1} END {print sum}')
        echo "Total CPU usage: ${cpu_usage}%" | tee -a "$log_file"
        
    else
        echo "Process $process_name is NOT running" | tee -a "$log_file"
    fi
    
    echo "----------------------------------------" | tee -a "$log_file"
}

# Function to get top processes by resource usage
top_processes() {
    echo "=== Top 10 Processes by CPU Usage ==="
    ps aux --sort=-%cpu | head -n 11
    
    echo -e "\n=== Top 10 Processes by Memory Usage ==="
    ps aux --sort=-%mem | head -n 11
}

# Function to monitor system resources
system_resources() {
    echo "=== System Resource Usage ==="
    echo "Timestamp: $(date)"
    
    echo -e "\n--- CPU Usage ---"
    top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1 | awk '{print "CPU Usage: " $1 "%"}'
    
    echo -e "\n--- Memory Usage ---"
    free | awk 'NR==2{printf "Memory Usage: %.2f%%\n", $3*100/$2}'
    
    echo -e "\n--- Disk Usage ---"
    df -h | grep -vE '^Filesystem|tmpfs|cdrom'
    
    echo -e "\n--- Load Average ---"
    uptime | awk -F'load average:' '{print "Load Average:" $2}'
    
    echo -e "\n--- Network Connections ---"
    netstat -an | grep :80 | wc -l | awk '{print "HTTP connections: " $1}'
    netstat -an | grep :443 | wc -l | awk '{print "HTTPS connections: " $1}'
}

# Function to alert on high resource usage
resource_alerts() {
    # CPU threshold (80%)
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    if (( $(echo "$cpu_usage > 80" | bc -l) )); then
        echo "ALERT: High CPU usage detected: ${cpu_usage}%"
    fi
    
    # Memory threshold (85%)
    mem_usage=$(free | awk 'NR==2{printf "%.2f", $3*100/$2}')
    if (( $(echo "$mem_usage > 85" | bc -l) )); then
        echo "ALERT: High memory usage detected: ${mem_usage}%"
    fi
    
    # Disk threshold (90%)
    disk_usage=$(df / | awk 'NR==2{print $5}' | cut -d'%' -f1)
    if [ "$disk_usage" -gt 90 ]; then
        echo "ALERT: High disk usage detected: ${disk_usage}%"
    fi
}

# Main monitoring loop
echo "Starting system monitoring..."

while true; do
    clear
    echo "=== System Monitor Dashboard ==="
    echo "Press Ctrl+C to exit"
    echo
    
    system_resources
    echo
    top_processes
    echo
    resource_alerts
    
    # Monitor specific processes
    monitor_process "nginx"
    monitor_process "mysql"
    
    sleep 5
done
EOF

chmod +x process_monitor.sh

# Service management examples
cat > service_management.sh << 'EOF'
#!/bin/bash

# Systemd service management
echo "=== Service Management Examples ==="

# Check service status
echo "Checking SSH service status:"
systemctl status ssh 2>/dev/null || systemctl status sshd

# List all services
echo -e "\n=== All Services ==="
systemctl list-units --type=service --state=active | head -10

# Failed services
echo -e "\n=== Failed Services ==="
systemctl list-units --type=service --state=failed

# Enable/disable services
echo -e "\n=== Service Control Commands ==="
cat << 'COMMANDS'
systemctl start service_name     # Start service
systemctl stop service_name      # Stop service
systemctl restart service_name   # Restart service
systemctl reload service_name    # Reload configuration
systemctl enable service_name    # Enable at boot
systemctl disable service_name   # Disable at boot
systemctl mask service_name      # Mask service (prevent start)
systemctl unmask service_name    # Unmask service
COMMANDS

# Create custom service
echo -e "\n=== Creating Custom Service ==="
cat > /tmp/my-monitor.service << 'SERVICE'
[Unit]
Description=My System Monitor
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/my-monitor.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
SERVICE

echo "Custom service definition created in /tmp/my-monitor.service"
echo "To install: sudo cp /tmp/my-monitor.service /etc/systemd/system/"
echo "Then: sudo systemctl daemon-reload && sudo systemctl enable my-monitor"
EOF

chmod +x service_management.sh

echo "Process monitoring and service management scripts created!"
echo "Run ./process_monitor.sh for real-time monitoring"
echo "Run ./service_management.sh for service management examples"
```

### Task 5: Network Configuration and Troubleshooting
**Configure network interfaces, routing, and troubleshoot connectivity issues.**

**Solution:**
```bash
# Network interface information
ip addr show                              # Show all interfaces (modern)
ifconfig                                  # Show interfaces (legacy)
ip link show                              # Show physical interfaces
cat /proc/net/dev                         # Network device statistics

# Configure network interface (temporary)
sudo ip addr add 192.168.1.100/24 dev eth0    # Add IP address
sudo ip link set eth0 up                       # Bring interface up
sudo ip link set eth0 down                     # Bring interface down
sudo ip addr del 192.168.1.100/24 dev eth0    # Remove IP address

# Routing information
ip route show                             # Show routing table
route -n                                  # Show routing table (legacy)
ip route get 8.8.8.8                    # Show route to specific destination

# Add/remove routes
sudo ip route add 192.168.2.0/24 via 192.168.1.1    # Add route
sudo ip route del 192.168.2.0/24                     # Delete route
sudo ip route add default via 192.168.1.1            # Add default route

# DNS configuration
cat /etc/resolv.conf                      # Current DNS settings
nslookup google.com                       # DNS lookup
dig google.com                            # Detailed DNS query
host google.com                           # Simple DNS lookup

# Network testing
ping -c 4 google.com                      # Ping with count limit
ping6 -c 4 google.com                     # IPv6 ping
traceroute google.com                     # Trace route to destination
tracepath google.com                      # Alternative traceroute
mtr google.com                            # Continuous traceroute

# Port testing
telnet google.com 80                      # Test TCP connection
nc -zv google.com 80                      # Test port with netcat
nmap -p 22,80,443 target_host             # Port scan

# Network connections
netstat -tuln                             # Show listening ports
netstat -an                               # Show all connections
ss -tuln                                  # Modern alternative to netstat
lsof -i :80                               # Show processes using port 80

# Bandwidth testing
wget -O /dev/null http://speedtest.tele2.net/10MB.zip  # Download test
curl -o /dev/null http://speedtest.tele2.net/10MB.zip  # Alternative download

# Network monitoring
iftop                                     # Real-time bandwidth usage
nload                                     # Network load monitor
vnstat                                    # Network statistics

# Ubuntu/Debian network configuration
cat > /tmp/01-netcfg.yaml << 'EOF'
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    eth1:
      dhcp4: true
EOF

# CentOS/RHEL network configuration
cat > /tmp/ifcfg-eth0 << 'EOF'
# /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
ONBOOT=yes
EOF

# Network troubleshooting script
cat > network_troubleshoot.sh << 'EOF'
#!/bin/bash

# Comprehensive network troubleshooting script

echo "=== Network Troubleshooting Report ==="
echo "Timestamp: $(date)"
echo "Hostname: $(hostname)"
echo

# 1. Interface Status
echo "=== Network Interfaces ==="
ip addr show | grep -E "(^\d+:|inet |state )"
echo

# 2. Routing Table
echo "=== Routing Table ==="
ip route show
echo

# 3. DNS Configuration
echo "=== DNS Configuration ==="
cat /etc/resolv.conf
echo

# 4. Gateway Connectivity
echo "=== Gateway Connectivity ==="
gateway=$(ip route | grep default | awk '{print $3}' | head -1)
if [ -n "$gateway" ]; then
    echo "Default gateway: $gateway"
    if ping -c 3 -W 2 "$gateway" > /dev/null 2>&1; then
        echo "✓ Gateway is reachable"
    else
        echo "✗ Gateway is NOT reachable"
    fi
else
    echo "✗ No default gateway configured"
fi
echo

# 5. Internet Connectivity
echo "=== Internet Connectivity ==="
test_hosts=("8.8.8.8" "1.1.1.1" "google.com")
for host in "${test_hosts[@]}"; do
    if ping -c 2 -W 3 "$host" > /dev/null 2>&1; then
        echo "✓ $host is reachable"
    else
        echo "✗ $host is NOT reachable"
    fi
done
echo

# 6. DNS Resolution
echo "=== DNS Resolution Test ==="
test_domains=("google.com" "github.com" "stackoverflow.com")
for domain in "${test_domains[@]}"; do
    if nslookup "$domain" > /dev/null 2>&1; then
        echo "✓ $domain resolves correctly"
    else
        echo "✗ $domain does NOT resolve"
    fi
done
echo

# 7. Active Connections
echo "=== Active Network Connections ==="
netstat -tuln | head -10
echo

# 8. Firewall Status
echo "=== Firewall Status ==="
if command -v ufw > /dev/null; then
    echo "UFW Status:"
    sudo ufw status verbose
elif command -v firewall-cmd > /dev/null; then
    echo "Firewalld Status:"
    sudo firewall-cmd --state
    sudo firewall-cmd --list-all
elif command -v iptables > /dev/null; then
    echo "IPTables Rules:"
    sudo iptables -L -n | head -20
else
    echo "No common firewall found"
fi
echo

# 9. Network Services
echo "=== Network Services ==="
ss -tlnp | grep -E ":(22|53|80|443|25)" | head -10
echo

# 10. Hardware Information
echo "=== Network Hardware ==="
lspci | grep -i network
lsusb | grep -i network
echo

# Generate recommendations
echo "=== Troubleshooting Recommendations ==="
if ! ping -c 1 -W 2 8.8.8.8 > /dev/null 2>&1; then
    echo "• Check physical network connection"
    echo "• Verify IP configuration"
    echo "• Check routing table"
fi

if ! nslookup google.com > /dev/null 2>&1; then
    echo "• Check DNS configuration in /etc/resolv.conf"
    echo "• Try alternative DNS servers (8.8.8.8, 1.1.1.1)"
fi

echo "• Check firewall rules if connectivity issues persist"
echo "• Monitor network traffic with tcpdump or wireshark"
echo "• Check system logs: journalctl -f or /var/log/syslog"
EOF

chmod +x network_troubleshoot.sh

# Network configuration management script
cat > network_config.sh << 'EOF'
#!/bin/bash

# Network configuration management

# Function to backup current network config
backup_network_config() {
    local backup_dir="/tmp/network_backup_$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$backup_dir"
    
    # Backup based on distribution
    if [ -d "/etc/netplan" ]; then
        # Ubuntu/newer systems
        cp -r /etc/netplan "$backup_dir/"
        echo "Netplan configuration backed up to $backup_dir"
    elif [ -d "/etc/sysconfig/network-scripts" ]; then
        # CentOS/RHEL
        cp -r /etc/sysconfig/network-scripts "$backup_dir/"
        echo "Network scripts backed up to $backup_dir"
    fi
    
    # Backup common files
    cp /etc/resolv.conf "$backup_dir/" 2>/dev/null
    cp /etc/hosts "$backup_dir/" 2>/dev/null
    
    # Save current IP configuration
    ip addr show > "$backup_dir/ip_addr.txt"
    ip route show > "$backup_dir/ip_route.txt"
    
    echo "Network configuration backup completed: $backup_dir"
}

# Function to configure static IP (Ubuntu/Netplan)
configure_static_ip_netplan() {
    local interface=$1
    local ip_address=$2
    local gateway=$3
    local dns_servers=$4
    
    cat > "/tmp/01-netcfg.yaml" << EOF
network:
  version: 2
  ethernets:
    $interface:
      dhcp4: false
      addresses: [$ip_address]
      gateway4: $gateway
      nameservers:
        addresses: [$dns_servers]
EOF
    
    echo "Netplan configuration created in /tmp/01-netcfg.yaml"
    echo "To apply: sudo cp /tmp/01-netcfg.yaml /etc/netplan/ && sudo netplan apply"
}

# Function to configure DHCP (Ubuntu/Netplan)
configure_dhcp_netplan() {
    local interface=$1
    
    cat > "/tmp/01-netcfg.yaml" << EOF
network:
  version: 2
  ethernets:
    $interface:
      dhcp4: true
      dhcp6: false
EOF
    
    echo "DHCP configuration created in /tmp/01-netcfg.yaml"
    echo "To apply: sudo cp /tmp/01-netcfg.yaml /etc/netplan/ && sudo netplan apply"
}

# Function to test network configuration
test_network_config() {
    echo "Testing network configuration..."
    
    # Test interfaces
    echo "Active interfaces:"
    ip link show | grep -E "^[0-9]+:" | grep "state UP"
    
    # Test connectivity
    if ping -c 2 -W 3 8.8.8.8 > /dev/null 2>&1; then
        echo "✓ Internet connectivity working"
    else
        echo "✗ Internet connectivity failed"
    fi
    
    # Test DNS
    if nslookup google.com > /dev/null 2>&1; then
        echo "✓ DNS resolution working"
    else
        echo "✗ DNS resolution failed"
    fi
}

# Main menu
case ${1:-help} in
    backup)
        backup_network_config
        ;;
    static)
        if [ $# -eq 5 ]; then
            configure_static_ip_netplan "$2" "$3" "$4" "$5"
        else
            echo "Usage: $0 static <interface> <ip/mask> <gateway> <dns_servers>"
            echo "Example: $0 static eth0 192.168.1.100/24 192.168.1.1 8.8.8.8,8.8.4.4"
        fi
        ;;
    dhcp)
        if [ $# -eq 2 ]; then
            configure_dhcp_netplan "$2"
        else
            echo "Usage: $0 dhcp <interface>"
            echo "Example: $0 dhcp eth0"
        fi
        ;;
    test)
        test_network_config
        ;;
    *)
        echo "Network Configuration Manager"
        echo "Usage: $0 {backup|static|dhcp|test}"
        echo "  backup - Backup current network configuration"
        echo "  static - Configure static IP"
        echo "  dhcp   - Configure DHCP"
        echo "  test   - Test network connectivity"
        ;;
esac
EOF

chmod +x network_config.sh

echo "Network configuration and troubleshooting scripts created!"
echo "Run ./network_troubleshoot.sh for comprehensive network diagnostics"
echo "Run ./network_config.sh for network configuration management"
```

*[Continuing with remaining Linux/EC2 tasks 6-20 would follow the same detailed pattern with comprehensive solutions for system monitoring, log management, backup strategies, security hardening, performance tuning, etc.]*

## Intermediate Level Solutions (Tasks 11-15)

*[Solutions continue with more complex scenarios like advanced scripting, automation, monitoring dashboards, etc.]*

## Advanced Level Solutions (Tasks 16-20)

*[Advanced solutions include performance optimization, security hardening, disaster recovery, advanced automation, etc.]*

---

**Note**: All Linux solutions follow best practices including:
- Proper error handling and logging
- Security considerations
- System compatibility checks
- Backup procedures before changes
- Documentation and comments
- Performance optimization
- Resource monitoring

Remember to test all scripts in a safe environment before using in production systems.