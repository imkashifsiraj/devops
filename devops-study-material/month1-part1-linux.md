# DevOps Upskilling Program — Complete Study Material
# Month 1: Foundations (Linux, Networking, Git, Shell Scripting)

---

# PART 1: LINUX ADMINISTRATION

---

## 1.1 What is Linux?

Linux is a free, open-source operating system kernel created by Linus Torvalds in 1991. Combined with GNU tools, it forms a complete OS used by 90%+ of cloud servers, all Android devices, and most supercomputers.

**Why DevOps Engineers MUST know Linux:**
- Almost all production servers run Linux
- Containers (Docker) are built on Linux kernel features
- Kubernetes nodes run Linux
- CI/CD agents typically run Linux
- Cloud instances default to Linux (cheaper, no licensing)
- Scripting and automation are native to Linux

**Common Distributions:**
- Ubuntu/Debian — Most popular for cloud, great community
- RHEL/CentOS/Rocky/AlmaLinux — Enterprise, used in banking/government (GCC)
- Amazon Linux — AWS default
- Alpine — Minimal, used in containers

---

## 1.2 File System Hierarchy

Everything in Linux is a file. Understanding the directory structure is fundamental.

```
/               Root directory — everything starts here
├── /bin        Essential command binaries (ls, cp, mv, cat)
├── /sbin       System binaries (fdisk, iptables, reboot)
├── /etc        Configuration files (CRITICAL — you'll edit these daily)
│   ├── /etc/nginx/         Nginx configuration
│   ├── /etc/ssh/           SSH server configuration
│   ├── /etc/hosts          Local DNS overrides
│   ├── /etc/resolv.conf    DNS nameserver configuration
│   ├── /etc/fstab          Filesystem mount table (persistent mounts)
│   ├── /etc/crontab        System cron jobs
│   ├── /etc/passwd         User account information
│   ├── /etc/shadow         Encrypted passwords
│   ├── /etc/group          Group definitions
│   ├── /etc/sudoers        Sudo access control
│   └── /etc/systemd/       Systemd service files
├── /home       User home directories (/home/username)
├── /root       Root user's home directory
├── /var        Variable data
│   ├── /var/log/           System and application logs
│   ├── /var/www/           Web server files (common)
│   └── /var/lib/           Application state data
├── /tmp        Temporary files (cleared on reboot!)
├── /opt        Optional/third-party software
├── /usr        User programs
│   ├── /usr/bin/           Non-essential user commands
│   ├── /usr/local/         Locally compiled software
│   └── /usr/share/         Shared data (docs, man pages)
├── /proc       Virtual filesystem — process & kernel info
│   ├── /proc/cpuinfo       CPU information
│   ├── /proc/meminfo       Memory information
│   └── /proc/[PID]/        Process-specific information
├── /dev        Device files
│   ├── /dev/sda            First hard disk
│   ├── /dev/null           Discards all data written to it
│   └── /dev/zero           Produces null bytes
├── /mnt        Temporary mount points
├── /media      Removable media mount points
└── /sys        Virtual filesystem — hardware info
```

**Pro Tip:** When you don't know where a config file is, use: `find /etc -name "*nginx*"` or `locate nginx.conf`

---

## 1.3 Essential Commands — Daily Usage

### Navigation & File Operations

```bash
# Where am I?
pwd                         # Print working directory

# Move around
cd /var/log                 # Go to absolute path
cd ..                       # Go up one directory
cd ~                        # Go to home directory
cd -                        # Go to previous directory

# List files
ls                          # Basic listing
ls -la                      # Long format + hidden files (MOST USED)
ls -lah                     # Human-readable sizes
ls -lt                      # Sort by modification time (newest first)
ls -lS                      # Sort by size (largest first)
ls -R                       # Recursive listing

# Create
mkdir -p /opt/app/config    # Create nested directories (-p = parents)
touch file.txt              # Create empty file or update timestamp

# Copy, Move, Delete
cp file.txt backup.txt      # Copy file
cp -r dir1/ dir2/           # Copy directory recursively
mv old.txt new.txt          # Rename/move file
rm file.txt                 # Delete file (NO RECYCLE BIN!)
rm -rf directory/           # Delete directory and contents (DANGEROUS)

# SAFETY TIP: Always use rm -ri for interactive confirmation
# Or alias: alias rm='rm -i' in ~/.bashrc
```

### Viewing & Searching File Contents

```bash
# View files
cat file.txt                # Print entire file
less file.txt               # Paginated view (q to quit, / to search)
head -20 file.txt           # First 20 lines
tail -50 file.txt           # Last 50 lines
tail -f /var/log/syslog     # FOLLOW log in real-time (Ctrl+C to stop)
tail -f log.txt | grep ERROR  # Follow only ERROR lines

# Search inside files
grep "error" /var/log/syslog           # Find "error" in file
grep -i "error" file.txt               # Case-insensitive
grep -r "password" /etc/               # Recursive search in directory
grep -n "TODO" *.py                    # Show line numbers
grep -v "DEBUG" app.log                # Exclude lines containing DEBUG
grep -c "ERROR" app.log                # Count matching lines
grep -E "error|warning|critical" log   # Multiple patterns (regex)

# Find files
find / -name "nginx.conf"              # Find by name
find /var/log -name "*.log" -mtime -1  # Logs modified in last 1 day
find / -size +100M                     # Files larger than 100MB
find / -user root -perm -4000          # Find SUID files (security)
find /tmp -mtime +7 -delete            # Delete files older than 7 days

# Which/Where
which docker                # Full path of a command
whereis nginx               # Find binary, source, man page
type kubectl                # How shell interprets a command
```

### Text Processing (Power Tools)

```bash
# awk — Column-based processing
awk '{print $1}' file.txt              # Print first column
awk -F: '{print $1}' /etc/passwd       # Print usernames (: delimiter)
awk '$3 > 1000' /etc/passwd            # Filter by column value
df -h | awk '{print $5, $6}'           # Print disk usage % and mount

# sed — Stream editor (find & replace)
sed 's/old/new/g' file.txt             # Replace all occurrences
sed -i 's/old/new/g' file.txt          # Edit file in-place
sed -n '10,20p' file.txt               # Print lines 10-20
sed '/^#/d' config.conf                # Delete comment lines

# cut — Extract columns
cut -d: -f1,3 /etc/passwd              # Cut fields 1 and 3
cut -d' ' -f1 access.log              # First column of access log

# sort & uniq
sort file.txt                          # Sort alphabetically
sort -n file.txt                       # Sort numerically
sort -r file.txt                       # Reverse sort
sort | uniq                            # Remove duplicates
sort | uniq -c | sort -rn              # Count occurrences, sort by frequency

# wc — Word/line count
wc -l file.txt                         # Count lines
cat access.log | wc -l                 # Count requests

# Combining tools (pipes) — THE POWER OF LINUX
# Find top 10 IP addresses in access log:
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Find processes using most memory:
ps aux | sort -k4 -rn | head -10

# Count errors per hour:
grep "ERROR" app.log | awk '{print $1, $2}' | cut -d: -f1-2 | sort | uniq -c
```

---

## 1.4 File Permissions & Ownership

```
Permission Structure:
  -rwxrwxrwx  1  owner  group  size  date  filename
  │└┬┘└┬┘└┬┘
  │ │   │   └── Others (everyone else)
  │ │   └────── Group
  │ └────────── Owner
  └──────────── File type (- = file, d = directory, l = symlink)

Permission Values:
  r (read)    = 4
  w (write)   = 2
  x (execute) = 1

Common Permission Sets:
  755 = rwxr-xr-x  → Executables, directories (owner full, others read+execute)
  644 = rw-r--r--  → Regular files (owner read+write, others read only)
  600 = rw-------  → Private files (SSH keys, secrets)
  700 = rwx------  → Private directories
  777 = rwxrwxrwx  → NEVER USE IN PRODUCTION (everyone can do everything)
```

```bash
# Change permissions
chmod 755 script.sh                    # Set exact permissions
chmod +x script.sh                     # Add execute for all
chmod u+x script.sh                    # Add execute for owner only
chmod go-w file.txt                    # Remove write from group & others
chmod -R 755 /var/www/                 # Recursive

# Change ownership
chown user:group file.txt              # Change owner and group
chown -R www-data:www-data /var/www/   # Recursive ownership change
chgrp developers project/              # Change group only

# Special permissions
chmod u+s binary                       # SUID — runs as file owner
chmod g+s directory                    # SGID — new files inherit group
chmod +t /tmp                          # Sticky bit — only owner can delete

# View permissions
ls -la                                 # List with permissions
stat file.txt                          # Detailed file information
getfacl file.txt                       # Access Control Lists (advanced)
```

**Real-World Scenario:**
```bash
# Web server setup — proper permissions
sudo chown -R www-data:www-data /var/www/myapp
sudo find /var/www/myapp -type d -exec chmod 755 {} \;   # Directories
sudo find /var/www/myapp -type f -exec chmod 644 {} \;   # Files
sudo chmod 600 /var/www/myapp/.env                        # Secrets file
```

---

## 1.5 User & Group Management

```bash
# Create users
useradd -m -s /bin/bash username       # Create with home dir and bash shell
useradd -m -G docker,sudo username     # Create with multiple groups
passwd username                        # Set password

# Modify users
usermod -aG docker username            # Add to group (IMPORTANT: -a = append)
usermod -L username                    # Lock account
usermod -U username                    # Unlock account
usermod -s /sbin/nologin username      # Disable shell access (service accounts)

# Delete users
userdel -r username                    # Delete user and home directory

# Groups
groupadd developers                    # Create group
groupdel developers                    # Delete group
groups username                        # Show user's groups
id username                            # Show UID, GID, groups

# Switch users
su - username                          # Switch to user (full login shell)
sudo -i                                # Become root
sudo -u username command               # Run command as another user

# Sudo configuration
visudo                                 # ALWAYS use visudo to edit sudoers
# In /etc/sudoers:
# username ALL=(ALL) NOPASSWD: ALL     # Full sudo without password
# %developers ALL=(ALL) ALL            # Group-based sudo access
```

**Best Practice for DevOps:**
```bash
# Create a deploy user (no password login, SSH only)
useradd -m -s /bin/bash deploy
mkdir -p /home/deploy/.ssh
chmod 700 /home/deploy/.ssh
# Add public key to authorized_keys
echo "ssh-ed25519 AAAA..." >> /home/deploy/.ssh/authorized_keys
chmod 600 /home/deploy/.ssh/authorized_keys
chown -R deploy:deploy /home/deploy/.ssh
# Give specific sudo rights
echo "deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp" >> /etc/sudoers.d/deploy
```

---

## 1.6 Process Management

```bash
# View processes
ps aux                                 # All processes (BSD syntax)
ps -ef                                 # All processes (System V syntax)
ps aux | grep nginx                    # Find specific process
pgrep -a nginx                         # Find process by name
top                                    # Real-time process monitor
htop                                   # Better interactive monitor (install it!)

# Process signals
kill PID                               # Send SIGTERM (graceful shutdown)
kill -9 PID                            # Send SIGKILL (force kill — last resort)
kill -HUP PID                          # Send SIGHUP (reload config)
killall nginx                          # Kill all processes by name
pkill -f "python app.py"              # Kill by command pattern

# Background processes
command &                              # Run in background
nohup command &                        # Run in background, survive logout
jobs                                   # List background jobs
fg %1                                  # Bring job 1 to foreground
disown %1                              # Detach job from terminal

# Process priority
nice -n 10 command                     # Start with lower priority
renice -n 5 -p PID                     # Change running process priority

# Resource usage
top -o %MEM                            # Sort by memory usage
ps aux --sort=-%cpu | head             # Top CPU consumers
ps aux --sort=-%mem | head             # Top memory consumers
```

---

## 1.7 Systemd & Service Management

Systemd is the modern init system and service manager on most Linux distributions.

```bash
# Service management
systemctl start nginx                  # Start service
systemctl stop nginx                   # Stop service
systemctl restart nginx                # Restart (stop + start)
systemctl reload nginx                 # Reload config without downtime
systemctl enable nginx                 # Start on boot
systemctl disable nginx                # Don't start on boot
systemctl status nginx                 # Check status (VERY USEFUL)
systemctl is-active nginx              # Check if running (for scripts)
systemctl is-enabled nginx             # Check if enabled on boot

# View logs (journalctl)
journalctl -u nginx                    # All logs for a service
journalctl -u nginx -f                 # Follow logs (like tail -f)
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since today
journalctl -u nginx -p err             # Only errors
journalctl -b                          # Logs since last boot
journalctl --disk-usage                # How much space logs use

# List services
systemctl list-units --type=service    # All loaded services
systemctl list-units --failed          # Failed services
```

**Creating a custom service:**
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=deploy
Group=deploy
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
ExecStop=/opt/myapp/stop.sh
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
Environment=NODE_ENV=production
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
```

```bash
# After creating/modifying service file:
sudo systemctl daemon-reload           # Reload systemd configuration
sudo systemctl enable myapp            # Enable on boot
sudo systemctl start myapp             # Start now
sudo systemctl status myapp            # Verify it's running
```

---

## 1.8 Package Management

```bash
# === Debian/Ubuntu (apt) ===
sudo apt update                        # Update package list (do this FIRST)
sudo apt upgrade                       # Upgrade all packages
sudo apt install nginx                 # Install package
sudo apt remove nginx                  # Remove (keep config)
sudo apt purge nginx                   # Remove + delete config
sudo apt autoremove                    # Remove unused dependencies
apt search keyword                     # Search for packages
apt show package                       # Package details
dpkg -l                                # List all installed packages
dpkg -l | grep nginx                   # Check if specific package installed

# === RHEL/CentOS/Rocky (yum/dnf) ===
sudo yum update                        # Update all packages
sudo yum install nginx                 # Install package
sudo yum remove nginx                  # Remove package
yum search keyword                     # Search packages
yum info package                       # Package details
rpm -qa                                # List all installed packages
rpm -qa | grep nginx                   # Check if installed

# === Universal ===
# Check what package provides a file:
dpkg -S /usr/bin/curl                  # Debian
rpm -qf /usr/bin/curl                  # RHEL
```

---

## 1.9 Disk & Storage Management

```bash
# Disk space
df -h                                  # Filesystem disk usage (human-readable)
df -ih                                 # Inode usage (when "no space" but df shows free)
du -sh /var/log/                       # Directory size
du -sh /* 2>/dev/null | sort -rh | head  # Largest directories at root
ncdu /                                 # Interactive disk usage (install it!)

# Find large files
find / -type f -size +100M -exec ls -lh {} \;  # Files > 100MB
find /var/log -name "*.gz" -mtime +30 -delete   # Delete old compressed logs

# Block devices
lsblk                                 # List block devices (disks, partitions)
fdisk -l                              # Detailed partition information
blkid                                 # Show filesystem UUIDs

# Mount filesystems
mount /dev/sdb1 /mnt/data             # Mount a partition
umount /mnt/data                      # Unmount
mount | grep sdb                      # Check what's mounted

# Persistent mounts (/etc/fstab)
# UUID=xxxxx /mnt/data ext4 defaults 0 2
# After editing fstab:
mount -a                              # Mount all entries in fstab

# Memory
free -h                               # Memory usage (human-readable)
# Output: total, used, free, shared, buff/cache, available
# "available" is what matters — it includes reclaimable cache

# Swap
swapon --show                         # Show swap usage
```

**Troubleshooting "Disk Full":**
```bash
# Step 1: Find what's full
df -h                                 # Which filesystem is 100%?

# Step 2: Find large files/directories
du -sh /var/* | sort -rh | head       # Largest dirs in /var
du -sh /var/log/* | sort -rh | head   # Usually logs are the culprit

# Step 3: Clean up
sudo journalctl --vacuum-size=500M    # Trim systemd logs
sudo find /var/log -name "*.gz" -mtime +7 -delete  # Old compressed logs
sudo apt clean                        # Clean package cache
docker system prune -a                # Clean Docker (if applicable)

# Step 4: Verify
df -h                                 # Confirm space reclaimed
```

---

## 1.10 SSH (Secure Shell)

SSH is how you access remote servers. Master this.

```bash
# Connect
ssh user@hostname                      # Basic connection
ssh -p 2222 user@hostname              # Custom port
ssh -i ~/.ssh/mykey.pem user@hostname  # Specific key

# Generate SSH keys (ALWAYS use this over passwords)
ssh-keygen -t ed25519 -C "your@email.com"
# Creates: ~/.ssh/id_ed25519 (private — NEVER share)
#          ~/.ssh/id_ed25519.pub (public — copy to servers)

# Copy public key to server
ssh-copy-id user@hostname              # Automatic method
# Manual method:
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# SSH Config file (~/.ssh/config) — SAVE TIME
# ~/.ssh/config
Host prod-web
    HostName 10.0.1.50
    User deploy
    IdentityFile ~/.ssh/prod-key
    Port 22

Host staging-*
    User deploy
    IdentityFile ~/.ssh/staging-key
    ProxyJump bastion

Host bastion
    HostName bastion.company.com
    User admin

# Now just: ssh prod-web (instead of full command)

# File transfer
scp file.txt user@host:/path/          # Copy file to remote
scp user@host:/path/file.txt .         # Copy from remote
scp -r directory/ user@host:/path/     # Copy directory

# Rsync (BETTER than scp — resume, delta transfer)
rsync -avz --progress ./files/ user@host:/destination/
rsync -avz --delete ./files/ user@host:/dest/  # Mirror (delete extra files on dest)

# SSH tunneling (port forwarding)
# Local forward: Access remote service through local port
ssh -L 8080:localhost:3000 user@host   # localhost:8080 → remote:3000
ssh -L 5432:db-server:5432 user@bastion  # Access DB through bastion

# Remote forward: Expose local service to remote
ssh -R 9090:localhost:3000 user@host   # remote:9090 → your localhost:3000
```

**SSH Security Best Practices:**
```bash
# /etc/ssh/sshd_config — Key settings:
PermitRootLogin no                     # Disable root SSH
PasswordAuthentication no              # Key-only authentication
Port 2222                             # Change default port (security through obscurity)
MaxAuthTries 3                         # Limit login attempts
AllowUsers deploy admin                # Whitelist users
```

---

## 1.11 Cron Jobs (Scheduled Tasks)

```bash
# Edit crontab
crontab -e                            # Edit current user's crontab
sudo crontab -u deploy -e             # Edit another user's crontab
crontab -l                            # List current cron jobs

# Cron format:
# ┌───── minute (0-59)
# │ ┌───── hour (0-23)
# │ │ ┌───── day of month (1-31)
# │ │ │ ┌───── month (1-12)
# │ │ │ │ ┌───── day of week (0-7, 0 and 7 = Sunday)
# │ │ │ │ │
# * * * * * command

# Examples:
*/5 * * * * /scripts/health-check.sh           # Every 5 minutes
0 2 * * * /scripts/backup.sh                    # Daily at 2:00 AM
0 0 * * 0 /scripts/weekly-cleanup.sh            # Every Sunday at midnight
0 9 * * 1-5 /scripts/workday-report.sh          # Weekdays at 9 AM
0 */6 * * * /scripts/sync-data.sh               # Every 6 hours
0 0 1 * * /scripts/monthly-report.sh            # First day of each month

# Best practices for cron:
# 1. Always use full paths
# 2. Redirect output to log file
# 3. Add error handling
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# 4. Use flock to prevent overlapping runs
*/5 * * * * /usr/bin/flock -n /tmp/check.lock /scripts/health-check.sh
```

**TIP:** Use https://crontab.guru to visualize cron expressions!

---

## 1.12 Log Management

```bash
# Important log locations
/var/log/syslog          # System messages (Debian/Ubuntu)
/var/log/messages        # System messages (RHEL/CentOS)
/var/log/auth.log        # Authentication logs (SSH logins, sudo)
/var/log/kern.log        # Kernel messages
/var/log/dmesg           # Boot messages
/var/log/nginx/          # Nginx access & error logs
/var/log/apt/            # Package management history

# Analyzing logs
tail -f /var/log/syslog | grep -i error     # Real-time errors
grep "Failed password" /var/log/auth.log    # Failed SSH attempts
zgrep "error" /var/log/syslog.*.gz          # Search compressed logs

# Log rotation (/etc/logrotate.d/)
# Example: /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data www-data
    postrotate
        systemctl reload myapp
    endscript
}
```

---

## 1.13 Performance Troubleshooting

When a server is slow, follow this methodology:

```bash
# Step 1: CPU
top                                    # Overall CPU usage
# Look for: %us (user), %sy (system), %wa (I/O wait)
# High %wa = disk bottleneck
mpstat -P ALL 1                        # Per-CPU stats

# Step 2: Memory
free -h                                # Overall memory
# If "available" is low → memory pressure
vmstat 1                               # Memory stats over time
# Check si/so (swap in/out) — if non-zero, server is swapping (BAD)

# Step 3: Disk I/O
iostat -x 1                            # Disk I/O stats
# Look for: %util > 80% → disk saturated
iotop                                  # Which process is doing I/O

# Step 4: Network
iftop                                  # Network traffic by connection
ss -s                                  # Socket statistics summary
netstat -an | grep ESTABLISHED | wc -l # Connection count

# Step 5: Open files / File descriptors
lsof | wc -l                           # Total open files
lsof -p PID                            # Files opened by process
ulimit -n                              # File descriptor limit
cat /proc/sys/fs/file-nr               # System-wide FD usage

# Quick diagnostic one-liner:
echo "=== LOAD ===" && uptime && echo "=== CPU ===" && top -bn1 | head -5 && echo "=== MEMORY ===" && free -h && echo "=== DISK ===" && df -h
```

---

## 1.14 Tips, Tricks & Productivity

```bash
# Keyboard shortcuts (HUGE time savers)
Ctrl + R        # Reverse search history (start typing to find old commands)
Ctrl + A        # Move cursor to beginning of line
Ctrl + E        # Move cursor to end of line
Ctrl + U        # Clear from cursor to beginning
Ctrl + K        # Clear from cursor to end
Ctrl + W        # Delete word before cursor
Ctrl + L        # Clear screen (same as 'clear')
!!              # Repeat last command
sudo !!         # Repeat last command with sudo
!$              # Last argument of previous command
Alt + .         # Insert last argument (cycle through history)

# Useful aliases (add to ~/.bashrc)
alias ll='ls -alh'
alias la='ls -A'
alias ..='cd ..'
alias ...='cd ../..'
alias grep='grep --color=auto'
alias ports='ss -tlnp'
alias myip='curl -s ifconfig.me'
alias disk='df -h | grep -v tmpfs'
alias psg='ps aux | grep'

# History tricks
history | tail -20                     # Recent commands
history | grep "docker"                # Find docker commands you ran
export HISTSIZE=10000                  # Keep more history
export HISTTIMEFORMAT="%Y-%m-%d %T "  # Add timestamps to history

# Screen/Tmux (terminal multiplexer — ESSENTIAL for remote work)
tmux new -s mysession                  # Create named session
tmux attach -t mysession               # Reattach to session
tmux ls                                # List sessions
# Inside tmux:
Ctrl+B then %                          # Split vertical
Ctrl+B then "                          # Split horizontal
Ctrl+B then d                          # Detach (session keeps running!)
Ctrl+B then c                          # New window
Ctrl+B then n/p                        # Next/previous window
```

---

## 1.15 Free Resources for Linux

| Resource | Link | Notes |
|----------|------|-------|
| Linux Journey | https://linuxjourney.com | Interactive beginner course |
| OverTheWire Bandit | https://overthewire.org/wargames/bandit/ | Learn Linux through CTF challenges |
| Linux Upskill Challenge | https://linuxupskillchallenge.org | 20-day server admin course |
| The Linux Command Line (book) | https://linuxcommand.org/tlcl.php | Free PDF book |
| ExplainShell | https://explainshell.com | Paste any command to understand it |
| KodeKloud Linux Basics | https://kodekloud.com | Free tier available |
| Red Hat System Admin (videos) | YouTube: "RHCSA" | RHEL-focused (GCC enterprise) |
| Ubuntu Server Guide | https://ubuntu.com/server/docs | Official documentation |
| Linux From Scratch | https://www.linuxfromscratch.org | Deep understanding (advanced) |

---
