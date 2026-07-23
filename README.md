# 🧠 DevOps Engineer — Second Brain

> **Purpose:** A comprehensive reference document covering everything a DevOps Engineer needs to know, from fundamentals to advanced concepts. Structured for easy reading, recall, and interview preparation.

> **How to use this document:**
> - Each topic follows: **What → Why → How → Key Concepts → Cheatsheet → Lessons Learned → Interview Questions**
> - Use the Table of Contents to jump to any section
> - Review one section per day for continuous learning

---

## 📋 Table of Contents

1. [Linux & Operating Systems](#1-linux--operating-systems)
2. [Networking Fundamentals](#2-networking-fundamentals)
3. [Git & Version Control](#3-git--version-control)
4. [Shell Scripting & Automation](#4-shell-scripting--automation)
5. [Docker & Containerization](#5-docker--containerization)
6. [Kubernetes (K8s)](#6-kubernetes-k8s)
7. [CI/CD Concepts](#7-cicd-concepts)
8. [GitLab CI/CD](#8-gitlab-cicd)
9. [Jenkins](#9-jenkins)
10. [GitHub Actions](#10-github-actions)
11. [Terraform (Infrastructure as Code)](#11-terraform-infrastructure-as-code)
12. [Ansible (Configuration Management)](#12-ansible-configuration-management)
13. [AWS Cloud](#13-aws-cloud)
14. [Azure Cloud](#14-azure-cloud)
15. [Monitoring & Observability](#15-monitoring--observability)
16. [Logging (ELK/EFK Stack)](#16-logging-elkefk-stack)
17. [Helm (Kubernetes Package Manager)](#17-helm-kubernetes-package-manager)
18. [Service Mesh (Istio)](#18-service-mesh-istio)
19. [Security & DevSecOps](#19-security--devsecops)
20. [Python for DevOps](#20-python-for-devops)
21. [Site Reliability Engineering (SRE)](#21-site-reliability-engineering-sre)
22. [ArgoCD & GitOps](#22-argocd--gitops)
23. [Databases for DevOps](#23-databases-for-devops)
24. [Soft Skills & Career Growth](#24-soft-skills--career-growth)

---

## 1. Linux & Operating Systems

### 📖 What Is It?
Linux is an open-source operating system kernel. It powers the majority of servers, cloud infrastructure, and containers in production environments. As a DevOps engineer, Linux is your daily workspace.

### 🎯 Why Is It Used?
- Free and open source — no licensing costs for servers
- Extremely stable and secure for production workloads
- Highly customizable and scriptable
- Dominant OS in cloud (90%+ of cloud workloads run Linux)
- Native container support (cgroups, namespaces)

### 🔧 How Is It Used?
You interact with Linux primarily through the terminal (CLI). You manage services, files, processes, users, networking, and storage.

### 🧩 Key Concepts

#### File System Hierarchy
```
/           → Root of everything
/home       → User home directories
/etc        → Configuration files
/var        → Variable data (logs, mail, spool)
/var/log    → System and application logs
/tmp        → Temporary files (cleared on reboot)
/opt        → Optional/third-party software
/usr        → User programs and utilities
/bin        → Essential command binaries
/sbin       → System administration binaries
/proc       → Virtual filesystem for process info
/dev        → Device files
/mnt        → Mount points for filesystems
```

#### File Permissions
```
rwx rwx rwx  →  owner | group | others
r=4, w=2, x=1

chmod 755 file    → rwxr-xr-x
chmod 644 file    → rw-r--r--
chmod +x script   → make executable
chown user:group file → change ownership
```

#### Process Management
```bash
ps aux              # List all processes
top / htop          # Real-time process monitoring
kill PID            # Send SIGTERM (graceful)
kill -9 PID         # Send SIGKILL (force)
systemctl start svc # Start a service
systemctl enable svc # Enable at boot
systemctl status svc # Check service status
journalctl -u svc   # View service logs
nohup command &     # Run in background, survive logout
```

#### User Management
```bash
useradd -m username     # Create user with home dir
usermod -aG group user  # Add user to group
passwd username         # Set password
su - username           # Switch user
sudo command            # Execute as root
visudo                  # Edit sudoers safely
```

#### Package Management
```bash
# Debian/Ubuntu
apt update && apt upgrade
apt install package
apt remove package
dpkg -l                  # List installed packages

# RHEL/CentOS
yum install package
dnf install package
rpm -qa                  # List installed packages
```

#### Disk & Storage
```bash
df -h                 # Disk usage (human readable)
du -sh /path          # Directory size
lsblk                 # List block devices
mount /dev/sdb1 /mnt  # Mount a filesystem
fdisk -l              # List partitions
free -h               # Memory usage
```

#### Important Files to Know
```
/etc/hosts          → Local DNS resolution
/etc/resolv.conf    → DNS server configuration
/etc/fstab          → Filesystem mount table (persistent mounts)
/etc/crontab        → System-wide cron jobs
/etc/ssh/sshd_config → SSH server configuration
/etc/passwd         → User accounts
/etc/shadow         → Encrypted passwords
/etc/group          → Group definitions
~/.bashrc           → User shell configuration
~/.ssh/             → SSH keys and config
```

#### Cron Jobs
```bash
crontab -e           # Edit user's cron
crontab -l           # List user's cron jobs

# Format: MIN HOUR DOM MON DOW COMMAND
# Examples:
0 2 * * * /backup.sh         # Daily at 2:00 AM
*/5 * * * * /health-check.sh # Every 5 minutes
0 0 * * 0 /weekly-task.sh    # Weekly on Sunday midnight
```

#### SSH
```bash
ssh user@host                    # Connect
ssh -i key.pem user@host         # Connect with key
ssh-keygen -t ed25519            # Generate key pair
ssh-copy-id user@host            # Copy public key to server
scp file user@host:/path         # Copy file to remote
rsync -avz src/ user@host:/dest/ # Sync files
```

### 📝 Cheatsheet — Daily Commands
```bash
# Navigation & Files
ls -la          # List all files with details
cd /path        # Change directory
pwd             # Print working directory
find / -name "file"  # Find files
grep -r "text" /path # Search text in files
tail -f /var/log/syslog  # Follow log in real-time
cat / less / more   # View file contents
wc -l file      # Count lines

# Text Processing
awk '{print $1}' file    # Print first column
sed 's/old/new/g' file   # Replace text
cut -d: -f1 /etc/passwd  # Cut by delimiter
sort | uniq              # Sort and deduplicate

# Networking (quick)
ip addr          # Show IP addresses
ss -tlnp         # Show listening ports
curl -v URL      # HTTP request with details
wget URL         # Download file
ping host        # Test connectivity
traceroute host  # Trace network path
```

### 💡 Lessons Learned
1. **Always use `systemctl` instead of `service`** on modern systems (systemd)
2. **Never run `rm -rf /`** — use safeguards, always double-check paths
3. **Use `screen` or `tmux`** for long-running tasks — SSH disconnects happen
4. **Check disk space FIRST** when services fail unexpectedly — full disks cause silent failures
5. **Set up SSH keys**, never rely on passwords for server access
6. **Log rotation matters** — unmanaged logs can fill disks in days
7. **Test cron jobs manually** before scheduling — cron environment differs from interactive shell
8. **`/tmp` gets cleared on reboot** — never store important data there

### ❓ Interview Questions
1. **Q: What is the difference between a process and a thread?**
   A: A process is an independent program with its own memory space. A thread is a lightweight unit within a process that shares the process's memory.

2. **Q: Explain Linux file permissions.**
   A: Three permission types (read/write/execute) for three categories (owner/group/others). Represented numerically (chmod 755) or symbolically (rwxr-xr-x).

3. **Q: What is a zombie process?**
   A: A process that has finished execution but still has an entry in the process table because its parent hasn't called wait() to read its exit status.

4. **Q: How do you check which port a service is running on?**
   A: `ss -tlnp`, `netstat -tlnp`, or `lsof -i :PORT`

5. **Q: What is the difference between hard link and soft link?**
   A: Hard link points to the same inode (same data), survives original deletion. Soft link (symlink) points to the filename, breaks if original is deleted.

6. **Q: How do you troubleshoot a server that's running slow?**
   A: Check CPU (`top`), memory (`free -h`), disk (`df -h`, `iostat`), network (`iftop`), processes (`ps aux --sort=-%cpu`), and logs (`/var/log/`).

7. **Q: What are cgroups and namespaces?**
   A: cgroups limit resource usage (CPU, memory) for process groups. Namespaces provide isolation (PID, network, mount). Together they are the foundation of containers.

---

## 2. Networking Fundamentals

### 📖 What Is It?
Networking is the backbone of all communication between systems. As a DevOps engineer, understanding how data flows between services, servers, and users is critical for troubleshooting, designing architectures, and ensuring reliability.

### 🎯 Why Is It Used?
- Every application communicates over a network
- Troubleshooting connectivity issues is a daily task
- Container networking (Docker, K8s) requires solid fundamentals
- Cloud architectures are built on virtual networking concepts
- Security depends on network controls (firewalls, ACLs, segmentation)

### 🔧 How Is It Used?
You configure networks, set up firewalls, troubleshoot connectivity, design VPCs, and ensure services can communicate securely.

### 🧩 Key Concepts

#### OSI Model (7 Layers) — Remember: "Please Do Not Throw Sausage Pizza Away"
```
Layer 7: Application   → HTTP, HTTPS, DNS, FTP, SSH, SMTP
Layer 6: Presentation  → SSL/TLS, encryption, data formatting
Layer 5: Session       → Session management, authentication
Layer 4: Transport     → TCP (reliable), UDP (fast)
Layer 3: Network       → IP addressing, routing
Layer 2: Data Link     → MAC addresses, switches, ARP
Layer 1: Physical      → Cables, signals, NICs
```

#### TCP vs UDP
```
TCP (Transmission Control Protocol):
- Connection-oriented (3-way handshake: SYN → SYN-ACK → ACK)
- Reliable delivery, ordered packets
- Used for: HTTP, SSH, FTP, databases

UDP (User Datagram Protocol):
- Connectionless
- No guarantee of delivery or order
- Used for: DNS, video streaming, gaming, VoIP
```

#### IP Addressing
```
IPv4: 32-bit, e.g., 192.168.1.100
IPv6: 128-bit, e.g., 2001:0db8:85a3::8a2e:0370:7334

Private IP Ranges (RFC 1918):
  10.0.0.0/8       → 10.0.0.0 - 10.255.255.255
  172.16.0.0/12    → 172.16.0.0 - 172.31.255.255
  192.168.0.0/16   → 192.168.0.0 - 192.168.255.255

CIDR Notation:
  /32 = 1 IP        (single host)
  /24 = 256 IPs     (common subnet)
  /16 = 65,536 IPs  (large network)
  /8  = 16M IPs     (massive network)

Subnet Mask Example:
  192.168.1.0/24 → Mask: 255.255.255.0
  Network: 192.168.1.0 | Hosts: .1-.254 | Broadcast: .255
```

#### DNS (Domain Name System)
```
How DNS Resolution Works:
  Browser → Local Cache → OS Cache → Router → 
  Recursive Resolver → Root NS → TLD NS → Authoritative NS → IP

Record Types:
  A      → Maps domain to IPv4 address
  AAAA   → Maps domain to IPv6 address
  CNAME  → Alias pointing to another domain
  MX     → Mail server
  NS     → Name server
  TXT    → Text records (SPF, DKIM, verification)
  SRV    → Service locator
  PTR    → Reverse DNS (IP → domain)
  SOA    → Start of Authority (zone info)

TTL (Time To Live):
  How long DNS record is cached
  Low TTL (60s) → Quick propagation, more DNS queries
  High TTL (86400s) → Less queries, slow propagation
```

#### Common Ports
```
22    → SSH
25    → SMTP
53    → DNS
80    → HTTP
443   → HTTPS
3306  → MySQL
5432  → PostgreSQL
6379  → Redis
8080  → HTTP Alt (common for apps)
8443  → HTTPS Alt
9090  → Prometheus
27017 → MongoDB
```

#### HTTP/HTTPS
```
HTTP Methods:
  GET    → Retrieve data
  POST   → Create/submit data
  PUT    → Update (full replacement)
  PATCH  → Partial update
  DELETE → Remove

Status Codes:
  1xx → Informational
  2xx → Success (200 OK, 201 Created, 204 No Content)
  3xx → Redirect (301 Permanent, 302 Temporary, 304 Not Modified)
  4xx → Client Error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found)
  5xx → Server Error (500 Internal, 502 Bad Gateway, 503 Service Unavailable)

HTTPS = HTTP + TLS
  TLS Handshake: ClientHello → ServerHello → Certificate → Key Exchange → Encrypted Session
```

#### Load Balancing
```
Types:
  Layer 4 (Transport) → Routes based on IP/port, fast, no content inspection
  Layer 7 (Application) → Routes based on HTTP content, headers, paths

Algorithms:
  Round Robin          → Equal distribution
  Least Connections   → Send to least busy server
  IP Hash             → Same client → same server (sticky)
  Weighted            → More traffic to stronger servers

Health Checks:
  - TCP check (is port open?)
  - HTTP check (does /health return 200?)
  - Custom script checks
```

#### Firewalls & Security Groups
```
iptables (Linux):
  iptables -L                       # List rules
  iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # Allow SSH
  iptables -A INPUT -j DROP         # Drop everything else

Cloud Security Groups (AWS):
  - Stateful (return traffic auto-allowed)
  - Inbound + Outbound rules
  - Attached to instances/ENIs

Network ACLs:
  - Stateless (must define both directions)
  - Subnet-level
  - Numbered rules evaluated in order
```

### 📝 Cheatsheet — Troubleshooting Commands
```bash
# Connectivity
ping host                    # Basic connectivity test
traceroute host              # Show path to destination
mtr host                     # Combines ping + traceroute
telnet host port             # Test if port is reachable
nc -zv host port             # Netcat port check

# DNS
nslookup domain              # Query DNS
dig domain                   # Detailed DNS query
dig +short domain A          # Quick A record lookup
host domain                  # Simple DNS lookup
cat /etc/resolv.conf         # Check DNS servers

# Network Interfaces
ip addr show                 # Show all interfaces & IPs
ip route show                # Show routing table
ifconfig                     # Legacy interface info

# Listening Services
ss -tlnp                     # Show listening TCP ports
ss -ulnp                     # Show listening UDP ports
lsof -i :8080               # What's using port 8080

# Traffic Analysis
tcpdump -i eth0 port 80     # Capture HTTP traffic
tcpdump -i any host 10.0.0.1 # Capture traffic to/from host
wireshark                    # GUI packet analysis

# HTTP Testing
curl -I https://example.com  # Get headers only
curl -v https://example.com  # Verbose (see TLS, headers)
curl -X POST -d 'data' URL  # POST request
wget --spider URL            # Check if URL exists
```

### 💡 Lessons Learned
1. **DNS is always the problem** — When in doubt, check DNS first (`dig`, `nslookup`)
2. **Security groups are stateful, NACLs are not** — This causes confusion in AWS
3. **Always check from both sides** — Can the client reach the server AND can the server respond?
4. **MTU mismatches** cause packet fragmentation and mysterious timeouts
5. **Port 443 blocked?** Many corporate networks only allow 80 and 443 outbound
6. **Use `curl -v`** to see the full connection lifecycle including TLS handshake
7. **Private DNS zones** in cloud can shadow public DNS — check which resolver you're hitting
8. **TCP keepalive** settings matter for long-lived connections through load balancers

### ❓ Interview Questions
1. **Q: What happens when you type google.com in a browser?**
   A: DNS resolution → TCP 3-way handshake → TLS handshake → HTTP GET request → Server processes → Response sent → Browser renders HTML/CSS/JS

2. **Q: Difference between TCP and UDP?**
   A: TCP is reliable, connection-oriented, guarantees delivery and order (HTTP, SSH). UDP is unreliable, connectionless, faster (DNS, streaming, gaming).

3. **Q: What is a subnet?**
   A: A logical subdivision of an IP network. Defined by CIDR notation (e.g., 10.0.1.0/24 = 256 addresses in that subnet).

4. **Q: How does a load balancer work?**
   A: Distributes incoming traffic across multiple backend servers using algorithms (round robin, least connections). Performs health checks to remove unhealthy servers.

5. **Q: What is NAT?**
   A: Network Address Translation — maps private IPs to a public IP for internet access. Used in VPCs for instances without public IPs.

6. **Q: How do you troubleshoot "connection refused" vs "connection timed out"?**
   A: "Refused" = port is closed or service is down (you reached the host). "Timed out" = can't reach the host at all (firewall, routing, host is down).

7. **Q: What is a VLAN vs a VPC?**
   A: VLAN is a Layer 2 broadcast domain in physical networking. VPC is a virtual private cloud network with subnets, route tables, and gateways in cloud environments.

---

## 3. Git & Version Control

### 📖 What Is It?
Git is a distributed version control system that tracks changes in source code. It allows multiple developers to collaborate, maintain history, and manage code releases.

### 🎯 Why Is It Used?
- Track every change ever made to code
- Enable collaboration without overwriting each other's work
- Support branching for parallel feature development
- Enable rollback to any previous state
- Foundation for CI/CD pipelines (code triggers builds)

### 🔧 How Is It Used?
Developers commit code locally, push to remote repositories (GitHub, GitLab, Bitbucket), create branches for features, and merge via pull/merge requests.

### 🧩 Key Concepts

#### Git Architecture
```
Working Directory → Staging Area (Index) → Local Repo → Remote Repo
     |                    |                     |              |
  git add            git commit            git push       (GitHub/GitLab)
     ←                   ←                    ←
  git checkout       git reset            git pull/fetch
```

#### Branching Strategies
```
Git Flow:
  main       → Production code
  develop    → Integration branch
  feature/*  → New features (branch from develop)
  release/*  → Preparation for release
  hotfix/*   → Emergency fixes (branch from main)

Trunk-Based Development:
  main       → Single branch, always deployable
  Short-lived feature branches (< 1 day)
  Feature flags for incomplete features

GitHub Flow:
  main       → Always deployable
  feature branches → PR → merge to main → deploy
```

#### Essential Commands
```bash
# Setup
git init                          # Initialize repo
git clone URL                     # Clone remote repo
git config --global user.name "Name"
git config --global user.email "email"

# Daily Workflow
git status                        # Check current state
git add file.txt                  # Stage specific file
git add .                         # Stage all changes
git commit -m "message"           # Commit with message
git push origin branch            # Push to remote
git pull origin branch            # Pull latest changes

# Branching
git branch                        # List branches
git branch feature-x              # Create branch
git checkout feature-x            # Switch to branch
git checkout -b feature-x         # Create + switch
git switch -c feature-x           # Modern create + switch
git merge feature-x               # Merge into current branch
git branch -d feature-x           # Delete branch (safe)
git branch -D feature-x           # Delete branch (force)

# History & Inspection
git log --oneline --graph         # Visual commit history
git log --author="name"           # Commits by author
git diff                          # Show unstaged changes
git diff --staged                 # Show staged changes
git show commit-hash              # Show specific commit
git blame file.txt                # Who changed each line

# Undoing Changes
git checkout -- file.txt          # Discard working changes
git reset HEAD file.txt           # Unstage a file
git reset --soft HEAD~1           # Undo commit, keep changes staged
git reset --hard HEAD~1           # Undo commit, discard changes ⚠️
git revert commit-hash            # Create new commit that undoes changes

# Stashing
git stash                         # Save current changes temporarily
git stash list                    # List all stashes
git stash pop                     # Apply and remove latest stash
git stash apply stash@{0}        # Apply specific stash

# Remote
git remote -v                     # List remotes
git fetch origin                  # Download without merging
git remote add upstream URL       # Add upstream remote

# Tags
git tag v1.0.0                    # Lightweight tag
git tag -a v1.0.0 -m "Release"   # Annotated tag
git push origin --tags            # Push all tags
```

#### Merge vs Rebase
```
Merge:
  - Creates a merge commit
  - Preserves complete history
  - Non-destructive
  - Use for: public/shared branches

Rebase:
  - Replays commits on top of target branch
  - Creates linear history (cleaner)
  - Rewrites history ⚠️
  - Use for: local/feature branches before merging
  - NEVER rebase public branches

Interactive Rebase (git rebase -i HEAD~3):
  pick   → Keep commit
  squash → Combine with previous commit
  reword → Change commit message
  drop   → Remove commit
```

#### .gitignore
```
# Common patterns
*.log              # All log files
node_modules/      # Dependencies
.env               # Environment variables (secrets!)
dist/              # Build output
*.pyc              # Python compiled files
.terraform/        # Terraform state cache
__pycache__/       # Python cache
.DS_Store          # Mac files
```

### 💡 Lessons Learned
1. **Never commit secrets** — Use .gitignore for .env files; use git-secrets or pre-commit hooks
2. **Write meaningful commit messages** — "fix bug" tells nothing; "Fix null pointer in user login when email is empty" is useful
3. **Never force push to shared branches** — It rewrites history and breaks teammates' work
4. **Pull before push** — Reduces merge conflicts
5. **Small, frequent commits** — Easier to review, easier to revert
6. **Use `.gitignore` from day one** — Adding it later means secrets may already be in history
7. **If a secret was committed**, consider it compromised — rotate it immediately, even after removing from history

### ❓ Interview Questions
1. **Q: What is the difference between `git merge` and `git rebase`?**
   A: Merge creates a merge commit preserving history. Rebase replays commits on top, creating linear history but rewriting commit hashes.

2. **Q: What is `git stash` used for?**
   A: Temporarily saves uncommitted changes so you can switch branches or pull updates, then re-apply them later.

3. **Q: How do you revert a commit that's already pushed?**
   A: Use `git revert <hash>` which creates a new commit that undoes the changes. Never use `reset --hard` on shared branches.

4. **Q: What is a detached HEAD state?**
   A: When HEAD points to a specific commit instead of a branch. Commits made here are "orphaned" unless you create a branch.

5. **Q: What is `git cherry-pick`?**
   A: Applies a specific commit from one branch onto another without merging the entire branch.

6. **Q: How do you remove a file from Git history?**
   A: Use `git filter-branch` or `BFG Repo Cleaner`. Note: this rewrites history and requires force push.

---

## 4. Shell Scripting & Automation

### 📖 What Is It?
Shell scripting is writing programs for the command-line shell (Bash) to automate repetitive tasks, orchestrate tools, and build operational workflows.

### 🎯 Why Is It Used?
- Automate repetitive manual tasks
- Build deployment and backup scripts
- Create health checks and monitoring scripts
- Glue together different tools and APIs
- First-line automation before reaching for Python or Ansible

### 🔧 How Is It Used?
Write `.sh` files with bash commands, logic, loops, and functions. Execute them manually, via cron, or as part of CI/CD pipelines.

### 🧩 Key Concepts

#### Script Structure
```bash
#!/bin/bash
# ^ Shebang — tells the system to use bash

set -euo pipefail
# -e: Exit on error
# -u: Error on undefined variables
# -o pipefail: Fail if any command in a pipe fails

# === Your script logic here ===
echo "Hello, DevOps!"
```

#### Variables
```bash
NAME="DevOps"               # No spaces around =
echo "Hello, $NAME"         # String interpolation
echo "Hello, ${NAME}!"      # Explicit boundary

# Command substitution
TODAY=$(date +%Y-%m-%d)
FILES=$(ls | wc -l)

# Environment variables
export MY_VAR="value"       # Available to child processes
echo "$HOME"                # User's home directory
echo "$PATH"                # Executable search path

# Special variables
$0   → Script name
$1   → First argument
$#   → Number of arguments
$@   → All arguments
$?   → Exit code of last command
$$   → Current process ID
```

#### Conditionals
```bash
# If statement
if [ "$ENV" == "production" ]; then
    echo "Running in production"
elif [ "$ENV" == "staging" ]; then
    echo "Running in staging"
else
    echo "Unknown environment"
fi

# File tests
[ -f file ]     # File exists
[ -d dir ]      # Directory exists
[ -r file ]     # File is readable
[ -w file ]     # File is writable
[ -x file ]     # File is executable
[ -s file ]     # File is not empty

# String tests
[ -z "$var" ]   # String is empty
[ -n "$var" ]   # String is not empty

# Numeric comparison
[ "$a" -eq "$b" ]  # Equal
[ "$a" -ne "$b" ]  # Not equal
[ "$a" -gt "$b" ]  # Greater than
[ "$a" -lt "$b" ]  # Less than
```

#### Loops
```bash
# For loop
for server in web1 web2 web3; do
    echo "Deploying to $server"
    ssh "$server" "sudo systemctl restart app"
done

# For loop with range
for i in {1..10}; do
    echo "Iteration $i"
done

# While loop
while [ "$count" -lt 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done

# Read file line by line
while IFS= read -r line; do
    echo "Processing: $line"
done < servers.txt
```

#### Functions
```bash
deploy() {
    local server=$1
    local version=$2
    echo "Deploying v${version} to ${server}..."
    ssh "$server" "docker pull app:${version} && docker restart app"
}

# Call function
deploy "web-server-01" "2.1.0"
```

#### Error Handling
```bash
# Trap errors
trap 'echo "Error on line $LINENO"; exit 1' ERR

# Check command success
if ! command -v docker &> /dev/null; then
    echo "Docker is not installed!"
    exit 1
fi

# Retry logic
retry() {
    local max_attempts=3
    local attempt=1
    until "$@"; do
        if [ "$attempt" -ge "$max_attempts" ]; then
            echo "Failed after $max_attempts attempts"
            return 1
        fi
        echo "Attempt $attempt failed. Retrying..."
        attempt=$((attempt + 1))
        sleep 5
    done
}
retry curl -f http://myapp.com/health
```

#### Practical Script Examples
```bash
# === Backup Script ===
#!/bin/bash
set -euo pipefail
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="production_db"

mysqldump "$DB_NAME" | gzip > "${BACKUP_DIR}/${DB_NAME}_${DATE}.sql.gz"

# Delete backups older than 7 days
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete
echo "Backup completed: ${DB_NAME}_${DATE}.sql.gz"

# === Health Check Script ===
#!/bin/bash
set -euo pipefail
SERVICES=("nginx" "app" "redis" "postgres")
FAILED=()

for svc in "${SERVICES[@]}"; do
    if ! systemctl is-active --quiet "$svc"; then
        FAILED+=("$svc")
    fi
done

if [ ${#FAILED[@]} -gt 0 ]; then
    echo "CRITICAL: Services down: ${FAILED[*]}"
    exit 2
fi
echo "OK: All services running"
```

### 💡 Lessons Learned
1. **Always use `set -euo pipefail`** — Catches errors that would silently continue
2. **Quote your variables** — `"$VAR"` not `$VAR` — prevents word splitting and globbing issues
3. **Use `shellcheck`** — Static analysis tool that catches common bash pitfalls
4. **Avoid parsing `ls` output** — Use `find` or globbing instead
5. **Use functions** — Makes scripts readable, testable, and reusable
6. **Log everything** — Include timestamps and context in script output
7. **Test with `bash -x script.sh`** — Shows every command as it executes (debug mode)
8. **Don't reinvent the wheel** — If a script gets >200 lines, consider Python or a proper tool

### ❓ Interview Questions
1. **Q: What does `set -euo pipefail` do?**
   A: `-e` exits on error, `-u` treats undefined variables as errors, `-o pipefail` fails if any command in a pipeline fails.

2. **Q: How do you pass arguments to a bash script?**
   A: Using positional parameters: `$1`, `$2`, etc. `$@` for all arguments, `$#` for count.

3. **Q: What is the difference between `$()` and backticks?**
   A: Both do command substitution. `$()` is preferred because it's nestable and more readable.

4. **Q: How do you make a script executable?**
   A: `chmod +x script.sh`, add shebang `#!/bin/bash` at top, then run with `./script.sh`.

5. **Q: What is a here document (heredoc)?**
   A: Multi-line string input: `cat <<EOF ... EOF`. Useful for embedding multi-line content in scripts.

---

## 5. Docker & Containerization

### 📖 What Is It?
Docker is a platform for packaging applications and their dependencies into lightweight, portable containers. A container is an isolated process that runs consistently across any environment.

### 🎯 Why Is It Used?
- **"Works on my machine" problem solved** — Same container runs everywhere
- Lightweight compared to VMs (shares host kernel)
- Fast startup (seconds vs minutes for VMs)
- Consistent environments (dev = staging = production)
- Microservices architecture enabler
- Immutable deployments — build once, deploy anywhere

### 🔧 How Is It Used?
Write a Dockerfile → Build an image → Push to registry → Pull and run containers in any environment.

### 🧩 Key Concepts

#### Containers vs VMs
```
Virtual Machines:
  Hardware → Hypervisor → Guest OS → App
  - Full OS per VM (GB in size)
  - Minutes to start
  - Strong isolation

Containers:
  Hardware → Host OS → Container Runtime → App
  - Shares host kernel (MB in size)
  - Seconds to start
  - Process-level isolation (namespaces + cgroups)
```

#### Docker Architecture
```
Docker Client (CLI) → Docker Daemon (dockerd) → Container Runtime (containerd → runc)
                                ↓
                    Images / Containers / Networks / Volumes

Registry (Docker Hub, ECR, GCR) ← push/pull → Docker Daemon
```

#### Dockerfile — Building Images
```dockerfile
# === Multi-stage build (production best practice) ===
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

#### Dockerfile Instructions
```dockerfile
FROM        # Base image (always first)
WORKDIR     # Set working directory
COPY        # Copy files from host to image
ADD         # Like COPY but can extract archives and fetch URLs
RUN         # Execute command during build
ENV         # Set environment variable
ARG         # Build-time variable
EXPOSE      # Document which port (does NOT publish)
VOLUME      # Create mount point
USER        # Set runtime user (security!)
HEALTHCHECK # Container health check
ENTRYPOINT  # Main executable (can't be overridden easily)
CMD         # Default arguments (can be overridden)
```

#### Docker Commands
```bash
# === Image Operations ===
docker build -t myapp:1.0 .            # Build image
docker build --no-cache -t myapp .     # Build without cache
docker images                          # List images
docker pull nginx:latest               # Pull from registry
docker push myrepo/myapp:1.0           # Push to registry
docker tag myapp:1.0 myrepo/myapp:1.0  # Tag image
docker rmi image_id                    # Remove image
docker image prune -a                  # Remove unused images

# === Container Operations ===
docker run -d --name myapp -p 8080:3000 myapp:1.0  # Run detached
docker run -it ubuntu:22.04 bash       # Interactive shell
docker run --rm myapp:1.0              # Auto-remove after exit
docker ps                              # List running containers
docker ps -a                           # List all (including stopped)
docker stop container_id               # Graceful stop
docker kill container_id               # Force stop
docker rm container_id                 # Remove container
docker exec -it container_id bash      # Shell into running container
docker logs -f container_id            # Follow logs
docker inspect container_id            # Detailed info (JSON)

# === Volumes (Persistent Data) ===
docker volume create mydata
docker run -v mydata:/app/data myapp   # Named volume
docker run -v /host/path:/container/path myapp  # Bind mount
docker volume ls                       # List volumes
docker volume prune                    # Remove unused volumes

# === Networks ===
docker network create mynet
docker run --network mynet --name app1 myapp
docker run --network mynet --name db postgres  # app1 can reach db by name
docker network ls
docker network inspect mynet

# === Docker Compose ===
docker compose up -d                   # Start all services
docker compose down                    # Stop and remove
docker compose logs -f service_name    # Follow service logs
docker compose build                   # Rebuild images
docker compose ps                      # List services
docker compose exec service bash       # Shell into service
```

#### Docker Compose Example
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: ${DB_PASSWORD}  # From .env file
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    networks:
      - backend

volumes:
  pgdata:

networks:
  backend:
    driver: bridge
```

#### Docker Networking
```
bridge (default)  → Containers on same host can communicate
host              → Container shares host network (no isolation)
none              → No networking
overlay           → Multi-host networking (Docker Swarm)
macvlan           → Container gets its own MAC address

Service Discovery in Compose:
  - Containers in same network reach each other by service name
  - e.g., app connects to DB via hostname "postgres"
```

#### Image Optimization Best Practices
```
1. Use small base images (alpine, distroless, slim)
2. Multi-stage builds (separate build & runtime)
3. Order Dockerfile for cache efficiency:
   - Things that change least → top (FROM, apt install)
   - Things that change most → bottom (COPY source code)
4. Combine RUN commands to reduce layers:
   RUN apt update && apt install -y pkg && rm -rf /var/lib/apt/lists/*
5. Use .dockerignore:
   node_modules
   .git
   *.md
   .env
6. Pin image versions (node:18.17.0-alpine, NOT node:latest)
7. Don't run as root — use USER instruction
```

### 💡 Lessons Learned
1. **Never use `latest` tag in production** — It's unpredictable; always pin versions
2. **Don't store data in containers** — Containers are ephemeral; use volumes for persistence
3. **One process per container** — Don't run multiple services in one container
4. **Use multi-stage builds** — Reduces image size dramatically (100s of MB saved)
5. **Scan images for vulnerabilities** — Use `docker scout`, Trivy, or Snyk
6. **Use `.dockerignore`** — Prevents copying unnecessary files into build context
7. **Run as non-root** — Critical security practice
8. **Health checks are essential** — Orchestrators use them to manage container lifecycle
9. **Layer caching is your friend** — Put rarely-changing instructions first
10. **`docker system prune`** — Regularly clean up to avoid filling disk

### ❓ Interview Questions
1. **Q: What is the difference between a Docker image and a container?**
   A: An image is a read-only template (blueprint). A container is a running instance of an image with a writable layer on top.

2. **Q: What is a multi-stage build?**
   A: A Dockerfile with multiple FROM statements. Build tools stay in early stages; only runtime artifacts are copied to the final image, reducing size.

3. **Q: Difference between CMD and ENTRYPOINT?**
   A: ENTRYPOINT defines the main executable (hard to override). CMD provides default arguments (easily overridden). Together: `ENTRYPOINT ["python"]` + `CMD ["app.py"]`.

4. **Q: How do you persist data in Docker?**
   A: Volumes (managed by Docker, best for production) or bind mounts (host directory mounted into container).

5. **Q: What is a Docker layer?**
   A: Each instruction in a Dockerfile creates a read-only layer. Layers are cached and reused, making rebuilds fast.

6. **Q: How do containers communicate with each other?**
   A: Through Docker networks. Containers on the same user-defined network can reach each other by container name (DNS).

7. **Q: How do you reduce Docker image size?**
   A: Alpine/distroless base images, multi-stage builds, minimize layers, use .dockerignore, remove package manager caches.

8. **Q: What is the difference between COPY and ADD?**
   A: COPY simply copies files. ADD can extract tar archives and fetch URLs. Best practice: use COPY unless you specifically need ADD's features.

---

## 6. Kubernetes (K8s)

### 📖 What Is It?
Kubernetes is an open-source container orchestration platform. It automates deployment, scaling, and management of containerized applications across clusters of machines.

### 🎯 Why Is It Used?
- Automates container deployment and scaling
- Self-healing (restarts failed containers, replaces unhealthy nodes)
- Service discovery and load balancing built-in
- Rolling updates with zero downtime
- Declarative configuration (desired state management)
- Works across cloud providers (portable)

### 🔧 How Is It Used?
Write YAML manifests describing desired state → Apply them with `kubectl` → Kubernetes ensures the cluster matches that state continuously.

### 🧩 Key Concepts

#### Architecture
```
Control Plane (Master):
  ├── API Server (kube-apiserver)     → Front door to the cluster, handles all requests
  ├── etcd                            → Key-value store for all cluster data
  ├── Scheduler (kube-scheduler)      → Assigns pods to nodes
  └── Controller Manager              → Runs control loops (ReplicaSet, Node, etc.)

Worker Nodes:
  ├── kubelet                         → Agent on each node, manages pods
  ├── kube-proxy                      → Handles networking rules (Service → Pod)
  └── Container Runtime               → Runs containers (containerd, CRI-O)
```

#### Core Resources
```
Pod           → Smallest deployable unit (1+ containers)
ReplicaSet    → Ensures N replicas of a pod are running
Deployment    → Manages ReplicaSets, handles rolling updates
Service       → Stable network endpoint for pods
Namespace     → Virtual cluster for isolation
ConfigMap     → Non-sensitive configuration data
Secret        → Sensitive data (base64 encoded, NOT encrypted by default)
PV/PVC        → Persistent storage
Ingress       → HTTP routing (virtual host, path-based)
DaemonSet     → One pod per node (logging agents, monitoring)
StatefulSet   → For stateful apps (databases) - stable identities
Job/CronJob   → Run-to-completion tasks / scheduled tasks
HPA           → Horizontal Pod Autoscaler
NetworkPolicy → Firewall rules between pods
ServiceAccount → Identity for pods to access API
```

#### Pod Lifecycle
```
Pending → Running → Succeeded/Failed

Pod States:
  Pending      → Waiting for scheduling or image pull
  Running      → At least one container running
  Succeeded    → All containers completed (Jobs)
  Failed       → All containers terminated, at least one failed
  Unknown      → Node communication lost

Container States:
  Waiting      → Not running (pulling image, etc.)
  Running      → Executing
  Terminated   → Finished (exit code available)
```

#### Essential YAML Manifests

```yaml
# === Deployment ===
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Extra pods during update
      maxUnavailable: 0    # Zero downtime
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.2.3
        ports:
        - containerPort: 8080
        resources:
          requests:          # Minimum guaranteed
            memory: "128Mi"
            cpu: "100m"
          limits:            # Maximum allowed
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:       # Restart if unhealthy
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 15
        readinessProbe:      # Remove from service if not ready
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: db_host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: db_password
        volumeMounts:
        - name: config-vol
          mountPath: /app/config
      volumes:
      - name: config-vol
        configMap:
          name: myapp-config

---
# === Service (ClusterIP) ===
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
# === Service Types ===
# ClusterIP   → Internal only (default)
# NodePort    → Expose on each node's IP:port (30000-32767)
# LoadBalancer → Cloud provider load balancer (external)
# ExternalName → Maps to external DNS name

---
# === Ingress ===
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80

---
# === HorizontalPodAutoscaler ===
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

---
# === ConfigMap ===
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  db_host: "postgres-service"
  log_level: "info"
  app.properties: |
    server.port=8080
    cache.ttl=300

---
# === Secret ===
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
data:
  db_password: cGFzc3dvcmQxMjM=  # base64 encoded
```

#### kubectl Commands
```bash
# === Cluster Info ===
kubectl cluster-info
kubectl get nodes
kubectl get namespaces

# === Core Operations ===
kubectl apply -f manifest.yaml         # Create/update resources
kubectl delete -f manifest.yaml        # Delete resources
kubectl get pods -n namespace          # List pods
kubectl get all -n namespace           # List everything
kubectl describe pod pod-name          # Detailed pod info
kubectl logs pod-name -f               # Follow pod logs
kubectl logs pod-name -c container     # Specific container logs
kubectl logs pod-name --previous       # Previous container logs (crash)
kubectl exec -it pod-name -- bash      # Shell into pod
kubectl port-forward pod-name 8080:80  # Local port forwarding

# === Debugging ===
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod pod-name          # Check Events section
kubectl get pod pod-name -o yaml       # Full pod spec
kubectl top pods                       # Resource usage
kubectl top nodes                      # Node resource usage

# === Deployments ===
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp           # Rollback to previous
kubectl rollout undo deployment/myapp --to-revision=2
kubectl scale deployment myapp --replicas=5
kubectl set image deployment/myapp myapp=myapp:2.0.0

# === Context & Config ===
kubectl config get-contexts            # List contexts
kubectl config use-context prod        # Switch context
kubectl config current-context         # Show current

# === Labels & Selectors ===
kubectl get pods -l app=myapp          # Filter by label
kubectl label pod pod-name env=prod    # Add label
```

#### Networking in Kubernetes
```
Pod-to-Pod:    Every pod gets an IP. Pods can reach each other directly.
Pod-to-Service: Service provides stable IP/DNS for a group of pods.
External-to-Service: Ingress/LoadBalancer exposes services externally.

DNS in Kubernetes:
  service-name.namespace.svc.cluster.local
  Example: postgres-service.production.svc.cluster.local

NetworkPolicy (firewall between pods):
  - Default: all pods can talk to all pods
  - NetworkPolicy restricts traffic (ingress/egress)
  - Must have a CNI that supports it (Calico, Cilium)
```

### 💡 Lessons Learned
1. **Always set resource requests AND limits** — Without them, one pod can starve others
2. **Use namespaces** for environment separation (dev, staging, prod)
3. **Never use `latest` tag** in deployments — Use specific versions for reproducibility
4. **Liveness ≠ Readiness probes** — Liveness restarts; Readiness removes from traffic
5. **Don't store state in pods** — Use StatefulSets + PVCs for databases, or external managed services
6. **`kubectl describe` is your best friend** — Check the Events section for why things fail
7. **Resource limits too tight = OOMKilled** — Monitor actual usage before setting limits
8. **Init containers** run before app containers — Great for DB migrations, waiting for dependencies
9. **Pod disruption budgets (PDB)** prevent all replicas from dying during node maintenance
10. **Secrets are base64, NOT encrypted** — Use External Secrets Operator or Sealed Secrets for real security

### ❓ Interview Questions
1. **Q: What is the difference between a Deployment and a StatefulSet?**
   A: Deployment for stateless apps (pods are interchangeable). StatefulSet for stateful apps — provides stable network identity, ordered creation/deletion, and persistent storage per pod.

2. **Q: How does a Service discover pods?**
   A: Through label selectors. The Service selector matches pod labels. kube-proxy/iptables routes traffic to matching pods.

3. **Q: What happens when a pod is OOMKilled?**
   A: The container exceeded its memory limit. Kubernetes kills it and restarts it (based on restartPolicy). Fix: increase memory limit or fix the memory leak.

4. **Q: Explain the difference between liveness and readiness probes.**
   A: Liveness: "Is this container alive?" Failure → container is restarted. Readiness: "Is this container ready to serve traffic?" Failure → removed from Service endpoints.

5. **Q: How do rolling updates work?**
   A: New pods are created with the new version while old pods are terminated gradually. `maxSurge` controls how many extra pods, `maxUnavailable` controls how many can be down.

6. **Q: What is an Ingress Controller?**
   A: A pod running a reverse proxy (NGINX, Traefik, HAProxy) that reads Ingress resources and configures routing rules. The Ingress resource itself does nothing without a controller.

7. **Q: How do you troubleshoot a pod stuck in Pending?**
   A: Check `kubectl describe pod` → Events section. Common causes: insufficient resources, node selector mismatch, PVC not bound, taints without tolerations.

8. **Q: What is a DaemonSet used for?**
   A: Ensures one pod runs on every node (or selected nodes). Used for: log collectors (Fluentd), monitoring agents (node-exporter), CNI plugins.

---

## 7. CI/CD Concepts

### 📖 What Is It?
CI/CD (Continuous Integration / Continuous Delivery / Continuous Deployment) is a set of practices that automate the process of integrating code changes, testing them, and delivering them to production.

### 🎯 Why Is It Used?
- Catch bugs early (automated testing on every commit)
- Reduce manual deployment errors
- Ship features faster with confidence
- Standardize the release process
- Enable rapid iteration and feedback loops

### 🔧 How Is It Used?
Developers push code → CI pipeline runs (build, test, scan) → If passed, CD pipeline deploys to environments (staging → production).

### 🧩 Key Concepts

#### CI vs CD
```
Continuous Integration (CI):
  - Merge code frequently (multiple times/day)
  - Automated build on every push
  - Automated tests (unit, integration)
  - Fast feedback to developers

Continuous Delivery (CD):
  - Code is always in a deployable state
  - Deployment to production requires manual approval
  - Automated deployment to staging/pre-prod

Continuous Deployment:
  - Every change that passes tests goes to production automatically
  - No manual gates
  - Requires high test confidence
```

#### Pipeline Stages (Typical)
```
1. Source       → Code pushed, pipeline triggered
2. Build        → Compile code, build Docker image
3. Unit Tests   → Fast tests, test individual components
4. Integration  → Test component interactions
5. Security     → SAST, dependency scanning, image scanning
6. Deploy Stage → Deploy to staging environment
7. E2E Tests    → End-to-end testing on staging
8. Approval     → Manual gate (for production)
9. Deploy Prod  → Deploy to production
10. Smoke Tests → Quick validation in production
```

#### Deployment Strategies
```
Rolling Update:
  - Replace old pods gradually with new ones
  - Zero downtime
  - Rollback by deploying previous version
  - Default in Kubernetes

Blue-Green Deployment:
  - Two identical environments (Blue = current, Green = new)
  - Switch traffic from Blue to Green once validated
  - Instant rollback (switch back to Blue)
  - Higher cost (double infrastructure)

Canary Deployment:
  - Route small % of traffic to new version (e.g., 5%)
  - Monitor for errors
  - Gradually increase traffic if healthy
  - Minimal blast radius

Feature Flags:
  - Deploy code to production with feature disabled
  - Enable for specific users/groups
  - Separate deployment from release
  - Tools: LaunchDarkly, Unleash, Flagsmith
```

#### Artifact Management
```
What are artifacts?
  - Built binaries, Docker images, packages
  - Should be built ONCE and promoted through environments
  - Never rebuild between environments

Registries:
  - Docker images: Docker Hub, ECR, GCR, Harbor
  - Packages: npm, PyPI, Maven Central
  - Generic: Artifactory, Nexus
```

### 💡 Lessons Learned
1. **Build once, deploy many** — Same artifact across all environments; only config changes
2. **Fast pipelines matter** — If CI takes 30 min, developers won't push frequently
3. **Tests are the backbone** — Without good tests, CI/CD is just "continuous deployment of bugs"
4. **Shift left** — Find issues early (security, quality) — cheaper to fix
5. **Rollback plan is mandatory** — Every deployment should have a tested rollback path
6. **Feature flags > long-lived branches** — Merge early, release when ready
7. **Pin your dependencies** — Unreliable builds from floating versions break pipelines
8. **Separate build and deploy** — Different concerns, different permissions

### ❓ Interview Questions
1. **Q: What is the difference between Continuous Delivery and Continuous Deployment?**
   A: Delivery means code is always deployable but requires manual approval for production. Deployment means every passing change goes to production automatically.

2. **Q: What is a canary deployment?**
   A: Releasing a new version to a small subset of users/traffic first, monitoring for issues, then gradually rolling out to all users.

3. **Q: What does "shift left" mean?**
   A: Moving quality checks (testing, security scanning) earlier in the development lifecycle where they're cheaper and faster to fix.

4. **Q: How do you handle database migrations in CI/CD?**
   A: Use migration tools (Flyway, Liquibase, Alembic). Migrations must be backward-compatible. Deploy migration before new code (expand-contract pattern).

---

## 8. GitLab CI/CD

### 📖 What Is It?
GitLab CI/CD is a built-in continuous integration and delivery tool within GitLab. Pipelines are defined in `.gitlab-ci.yml` at the repository root.

### 🎯 Why Is It Used?
- Fully integrated with GitLab (no separate tool to manage)
- Pipelines as code (versioned with your app)
- Built-in container registry, artifact storage, environments
- Supports complex pipelines (DAG, multi-project, parent-child)
- Auto DevOps for zero-config pipelines

### 🔧 How Is It Used?
Create `.gitlab-ci.yml` → Define stages and jobs → Push code → GitLab Runners execute jobs.

### 🧩 Key Concepts

#### Pipeline Structure
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# === Build Stage ===
build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main
    - merge_requests

# === Test Stage ===
unit-tests:
  stage: test
  image: node:18-alpine
  script:
    - npm ci
    - npm run test:unit
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: test-results.xml
    when: always

integration-tests:
  stage: test
  image: node:18-alpine
  services:
    - postgres:15-alpine
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
    DB_HOST: postgres
  script:
    - npm ci
    - npm run test:integration

# === Security Stage ===
sast:
  stage: security
  include:
    - template: Security/SAST.gitlab-ci.yml

container-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_IMAGE

# === Deploy Stages ===
deploy-staging:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.myapp.com
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n staging
    - kubectl rollout status deployment/myapp -n staging
  only:
    - main

deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://myapp.com
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n production
    - kubectl rollout status deployment/myapp -n production
  only:
    - main
  when: manual  # Manual approval gate
```

#### Key Features
```yaml
# --- Caching (speed up pipelines) ---
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/

# --- Artifacts (pass between jobs) ---
artifacts:
  paths:
    - dist/
    - coverage/
  expire_in: 1 week

# --- Rules (conditional execution) ---
deploy:
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: always
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
    - when: never

# --- Environments & Review Apps ---
review:
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.myapp.com
    on_stop: stop-review
  script:
    - deploy_review_app

# --- Templates & Includes ---
include:
  - local: '/ci/templates/.build.yml'
  - project: 'devops/ci-templates'
    ref: main
    file: '/templates/deploy.yml'
  - template: Security/SAST.gitlab-ci.yml

# --- Parent-Child Pipelines ---
trigger-deploy:
  trigger:
    include: deploy/.gitlab-ci.yml
    strategy: depend
```

#### Predefined CI/CD Variables
```
$CI_COMMIT_SHA              → Full commit hash
$CI_COMMIT_SHORT_SHA        → Short commit hash (8 chars)
$CI_COMMIT_BRANCH           → Branch name
$CI_COMMIT_REF_SLUG         → URL-safe branch name
$CI_PIPELINE_ID             → Pipeline ID
$CI_JOB_ID                  → Job ID
$CI_REGISTRY                → Container registry URL
$CI_REGISTRY_IMAGE          → Image path in registry
$CI_PROJECT_NAME            → Project name
$CI_ENVIRONMENT_NAME        → Environment name
$CI_MERGE_REQUEST_IID       → MR number
```

### 💡 Lessons Learned
1. **Use `rules:` instead of `only/except`** — More powerful and easier to understand
2. **Cache dependencies** — `node_modules`, `.m2`, pip cache — saves minutes per pipeline
3. **Use job templates** with `extends:` to reduce duplication
4. **Keep pipelines under 10 minutes** — Parallelize where possible
5. **Use `needs:` (DAG)** to run jobs as soon as dependencies are met (not just stage order)
6. **Store secrets in CI/CD Variables** (masked + protected), never in `.gitlab-ci.yml`
7. **Review apps** give each MR a live preview environment — invaluable for team review
8. **Use `interruptible: true`** for jobs that can be safely canceled when new commits come in

### ❓ Interview Questions
1. **Q: What is the difference between `cache` and `artifacts`?**
   A: Cache speeds up pipelines by reusing files between pipeline runs (e.g., dependencies). Artifacts pass files between jobs within the same pipeline and can be downloaded.

2. **Q: How do you run a job only on merge requests?**
   A: `rules: - if: $CI_PIPELINE_SOURCE == "merge_request_event"`

3. **Q: What are GitLab Runners?**
   A: Agents that execute CI/CD jobs. Can be shared (GitLab-hosted) or self-managed. Run jobs in Docker containers, VMs, or directly on the host.

4. **Q: How do you handle secrets in GitLab CI?**
   A: Store in Settings → CI/CD → Variables. Mark as "masked" (hidden in logs) and "protected" (only available on protected branches).

---

## 9. Jenkins

### 📖 What Is It?
Jenkins is an open-source automation server for building CI/CD pipelines. It uses a plugin architecture and Groovy-based Jenkinsfiles for pipeline definitions.

### 🎯 Why Is It Used?
- Extremely flexible with 1800+ plugins
- Self-hosted (full control over infrastructure)
- Supports any language, tool, or platform
- Large community and enterprise adoption
- Free and open source

### 🔧 How Is It Used?
Install Jenkins → Configure nodes/agents → Create pipeline jobs with Jenkinsfile → Code push triggers pipeline.

### 🧩 Key Concepts

#### Jenkinsfile (Declarative Pipeline)
```groovy
pipeline {
    agent {
        docker { image 'node:18-alpine' }
    }

    environment {
        REGISTRY = 'my-registry.com'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                    post {
                        always {
                            junit 'test-results/*.xml'
                        }
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-registry',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "docker login -u $USER -p $PASS ${REGISTRY}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh "kubectl set image deployment/myapp myapp=${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to production?"
                ok "Yes, deploy!"
            }
            steps {
                sh "kubectl set image deployment/myapp myapp=${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} -n production"
            }
        }
    }

    post {
        success {
            slackSend(color: 'good', message: "✅ Build #${BUILD_NUMBER} succeeded")
        }
        failure {
            slackSend(color: 'danger', message: "❌ Build #${BUILD_NUMBER} failed")
        }
        always {
            cleanWs()  // Clean workspace
        }
    }
}
```

#### Jenkins Architecture
```
Jenkins Controller (Master):
  - Schedules jobs
  - Manages configuration
  - Serves UI
  - Should NOT run builds (delegate to agents)

Jenkins Agents (Nodes):
  - Execute build jobs
  - Can be static (always running) or dynamic (cloud-provisioned)
  - Connect via SSH or JNLP
  - Labels define capabilities (linux, docker, gpu)
```

### 💡 Lessons Learned
1. **Use declarative pipelines** over scripted — clearer structure, better error messages
2. **Jenkins controller should not run builds** — Use agents for all workloads
3. **Shared libraries** — Extract common pipeline logic into shared libraries for reuse
4. **Credentials plugin** — Never hardcode secrets; use Jenkins credential store
5. **Pipeline as code** — Jenkinsfile in the repo, not configured in UI
6. **Backup Jenkins config** — `$JENKINS_HOME` is your entire CI/CD — back it up
7. **Jenkins can become a bottleneck** — Consider modern alternatives (GitLab CI, GitHub Actions) for new projects

### ❓ Interview Questions
1. **Q: Declarative vs Scripted pipeline?**
   A: Declarative has predefined structure (`pipeline { stages { } }`), easier to read. Scripted uses full Groovy, more flexible but harder to maintain.

2. **Q: How do you handle credentials in Jenkins?**
   A: Jenkins Credentials plugin stores secrets. Access in pipeline via `withCredentials()` block. Never echo credentials.

3. **Q: What is a Jenkins Shared Library?**
   A: A Git repo with reusable Groovy code that multiple pipelines can import. Reduces duplication across Jenkinsfiles.

---

## 10. GitHub Actions

### 📖 What Is It?
GitHub Actions is GitHub's native CI/CD platform. Workflows are defined in YAML files under `.github/workflows/`.

### 🎯 Why Is It Used?
- Native GitHub integration (no external tool needed)
- Marketplace with thousands of reusable actions
- Matrix builds for multi-platform testing
- Free for public repos; generous free tier for private
- Simple YAML syntax

### 🔧 How Is It Used?
Create workflow YAML → Define triggers and jobs → Push code → GitHub runs the workflow on hosted or self-hosted runners.

### 🧩 Key Concepts

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:  # Manual trigger

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]  # Test multiple versions
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test

  build-and-push:
    needs: test  # Depends on test job
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production  # Requires approval
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp myapp=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

### 💡 Lessons Learned
1. **Use specific action versions** (`@v4` not `@main`) — Prevents supply chain attacks
2. **Cache dependencies** — `actions/cache` or built-in cache in setup actions
3. **Use `GITHUB_TOKEN`** where possible — Auto-generated, scoped, no manual secret management
4. **Reusable workflows** — Extract common patterns for organization-wide reuse
5. **Self-hosted runners** for private repos with heavy workloads (save costs)

---

## 11. Terraform (Infrastructure as Code)

### 📖 What Is It?
Terraform is an Infrastructure as Code (IaC) tool by HashiCorp. It lets you define cloud infrastructure in declarative configuration files, manage state, and provision resources across any cloud provider.

### 🎯 Why Is It Used?
- **Infrastructure as Code** — Version-controlled, reviewable, repeatable
- Multi-cloud support (AWS, Azure, GCP, Kubernetes, etc.)
- Declarative — Describe desired state, Terraform figures out how to get there
- State management — Knows what exists, what changed
- Plan before apply — Preview changes before making them
- Modular and reusable

### 🔧 How Is It Used?
Write `.tf` files → `terraform init` → `terraform plan` → `terraform apply` → Infrastructure created/updated.

### 🧩 Key Concepts

#### Terraform Workflow
```
Write → Init → Plan → Apply → Destroy

terraform init      # Download providers & modules, initialize backend
terraform plan      # Preview what will change (dry run)
terraform apply     # Create/modify infrastructure
terraform destroy   # Destroy all managed infrastructure
terraform fmt       # Format code consistently
terraform validate  # Check syntax
terraform state list # List resources in state
terraform output    # Show output values
terraform import    # Import existing infrastructure into state
```

#### HCL (HashiCorp Configuration Language)
```hcl
# === Provider Configuration ===
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  # Remote state backend
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "eu-west-1"
    dynamodb_table = "terraform-locks"  # State locking
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  }
}
```

#### Resources, Variables, Outputs
```hcl
# === Variables (variables.tf) ===
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

# === Resources (main.tf) ===
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.project_name}-vpc"
  }
}

resource "aws_subnet" "public" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = var.availability_zones[count.index]

  map_public_ip_on_launch = true

  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
  }
}

resource "aws_security_group" "web" {
  name_prefix = "${var.project_name}-web-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_instance" "web" {
  count                  = var.instance_count
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public[count.index % length(aws_subnet.public)].id
  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y docker
    systemctl start docker
    systemctl enable docker
  EOF

  tags = {
    Name = "${var.project_name}-web-${count.index + 1}"
  }
}

# === Data Sources ===
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

# === Outputs (outputs.tf) ===
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "instance_ips" {
  description = "Public IPs of web instances"
  value       = aws_instance.web[*].public_ip
}
```

#### Modules (Reusable Components)
```hcl
# === Using a module ===
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"

  name = "${var.project_name}-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = var.environment != "prod"  # Save cost in non-prod
}

# === Custom module structure ===
# modules/
#   web-app/
#     main.tf       → Resources
#     variables.tf  → Input variables
#     outputs.tf    → Output values
#     README.md     → Documentation
```

#### State Management
```
What is Terraform State?
  - JSON file tracking all managed resources
  - Maps config to real-world resources
  - Stores resource IDs, attributes, dependencies

Remote State (ALWAYS use in teams):
  - S3 + DynamoDB (AWS)
  - Azure Blob Storage
  - GCS (Google Cloud)
  - Terraform Cloud

State Locking:
  - Prevents concurrent modifications
  - DynamoDB table for AWS backend
  - Automatic with most remote backends

State Commands:
  terraform state list              # List all resources
  terraform state show resource     # Show resource details
  terraform state mv old new        # Rename/move resource
  terraform state rm resource       # Remove from state (doesn't destroy)
  terraform import resource id      # Import existing resource
```

#### Terraform Best Practices
```
1. Remote state with locking (ALWAYS)
2. Use workspaces OR separate state files per environment
3. Use modules for reusable components
4. Pin provider versions
5. Use terraform fmt and terraform validate in CI
6. Never edit state manually
7. Use data sources for existing resources
8. Implement lifecycle rules where needed:
   - create_before_destroy (for zero-downtime)
   - prevent_destroy (for critical resources)
   - ignore_changes (for externally managed attributes)
9. Tag everything
10. Use -target sparingly (only for debugging)
```

### 💡 Lessons Learned
1. **Never commit state files** — Contains secrets, use remote backend
2. **Always run `plan` before `apply`** — Especially in production
3. **State drift happens** — Someone changes things in the console; use `terraform plan` to detect
4. **Modules save time** but add complexity — Don't over-modularize early
5. **`terraform destroy` is irreversible** — Be extremely careful, especially with databases
6. **Import existing infrastructure early** — Migrating later is painful
7. **Use `count` vs `for_each`**: `for_each` with maps is more stable (no index shifting)
8. **Separate state per environment** — One state file for prod, one for staging
9. **CI/CD for Terraform** — Plan on PR, apply on merge to main (with approval)
10. **`-target` is a crutch** — If you need it often, your structure needs refactoring

### ❓ Interview Questions
1. **Q: What is Terraform state and why is it important?**
   A: State is a JSON file mapping your configuration to real infrastructure. It tracks what Terraform manages, enables plan/diff, and stores resource metadata. Without it, Terraform can't know what exists.

2. **Q: How do you handle secrets in Terraform?**
   A: Mark variables as `sensitive = true`. Use remote state with encryption. Never commit `.tfstate`. Use vault providers or environment variables for sensitive inputs.

3. **Q: What is the difference between `count` and `for_each`?**
   A: `count` uses integer index (adding/removing items shifts all indices). `for_each` uses map keys (stable, adding/removing doesn't affect others). Prefer `for_each`.

4. **Q: How do you import existing infrastructure?**
   A: Write the resource block first, then `terraform import <resource_address> <real_world_id>`. Then run `plan` to ensure configuration matches reality.

5. **Q: What happens if two people run `terraform apply` simultaneously?**
   A: With state locking (DynamoDB), the second person gets a lock error. Without locking, state corruption can occur.

6. **Q: How do you manage multiple environments?**
   A: Separate state files per environment (different backend keys), use tfvars files per environment, or use Terraform workspaces (less recommended for large differences).

7. **Q: What is a data source?**
   A: A way to read information from existing infrastructure without managing it. Example: `data "aws_ami"` to find the latest AMI ID.

---

## 12. Ansible (Configuration Management)

### 📖 What Is It?
Ansible is an agentless configuration management and automation tool. It uses SSH to connect to servers and YAML playbooks to define desired states.

### 🎯 Why Is It Used?
- **Agentless** — No software to install on managed hosts (just SSH + Python)
- Simple YAML syntax (easy to read and write)
- Idempotent — Run multiple times, same result
- Configuration management (ensure servers are in desired state)
- Application deployment and orchestration
- Huge module library (3000+ modules)

### 🔧 How Is It Used?
Define inventory (which servers) → Write playbooks (what to do) → Run with `ansible-playbook` → Ansible connects via SSH and applies changes.

### 🧩 Key Concepts

#### Inventory
```ini
# inventory/hosts.ini
[webservers]
web1.example.com ansible_user=deploy
web2.example.com ansible_user=deploy

[databases]
db1.example.com ansible_user=admin

[production:children]
webservers
databases

[webservers:vars]
http_port=8080
```

```yaml
# inventory/hosts.yml (YAML format)
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_user: deploy
        web2.example.com:
          ansible_user: deploy
      vars:
        http_port: 8080
    databases:
      hosts:
        db1.example.com:
```

#### Playbook
```yaml
# playbooks/deploy-app.yml
---
- name: Deploy web application
  hosts: webservers
  become: yes  # sudo
  vars:
    app_version: "2.1.0"
    app_dir: /opt/myapp

  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

  tasks:
    - name: Install required packages
      apt:
        name:
          - docker.io
          - docker-compose
          - python3-pip
        state: present

    - name: Ensure Docker is running
      systemd:
        name: docker
        state: started
        enabled: yes

    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: deploy
        mode: '0755'

    - name: Copy Docker Compose file
      template:
        src: templates/docker-compose.yml.j2
        dest: "{{ app_dir }}/docker-compose.yml"
        owner: deploy
        mode: '0644'
      notify: Restart application

    - name: Deploy application
      community.docker.docker_compose:
        project_src: "{{ app_dir }}"
        state: present
        pull: yes

    - name: Wait for application to be healthy
      uri:
        url: "http://localhost:{{ http_port }}/health"
        status_code: 200
      register: health_check
      until: health_check.status == 200
      retries: 10
      delay: 5

  handlers:
    - name: Restart application
      community.docker.docker_compose:
        project_src: "{{ app_dir }}"
        state: present
        restarted: yes
```

#### Roles (Reusable Components)
```
roles/
  webserver/
    ├── tasks/main.yml       → Main task list
    ├── handlers/main.yml    → Event handlers
    ├── templates/           → Jinja2 templates
    ├── files/               → Static files
    ├── vars/main.yml        → Role variables
    ├── defaults/main.yml    → Default values (lowest priority)
    └── meta/main.yml        → Dependencies

# Using roles in playbook:
- name: Configure web servers
  hosts: webservers
  roles:
    - common
    - webserver
    - { role: ssl, when: ssl_enabled }
```

#### Common Modules
```yaml
# File operations
- file: path=/opt/app state=directory mode='0755'
- copy: src=app.conf dest=/etc/app.conf
- template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf

# Packages
- apt: name=nginx state=present
- yum: name=httpd state=latest

# Services
- systemd: name=nginx state=restarted enabled=yes

# Commands
- command: /opt/app/migrate.sh    # Simple command
- shell: cat /etc/hosts | grep db  # Shell features needed
- script: scripts/setup.sh         # Run local script on remote

# Users & Groups
- user: name=deploy groups=docker shell=/bin/bash
- authorized_key: user=deploy key="{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

# Docker
- docker_image: name=myapp tag=latest source=pull
- docker_container: name=myapp image=myapp:latest ports=["8080:80"]
```

#### Ansible Commands
```bash
# Run playbook
ansible-playbook -i inventory/hosts playbook.yml

# Limit to specific hosts
ansible-playbook playbook.yml -l webservers

# Dry run (check mode)
ansible-playbook playbook.yml --check --diff

# Run ad-hoc command
ansible all -i inventory -m ping
ansible webservers -m shell -a "free -h"
ansible webservers -m apt -a "name=nginx state=present" --become

# Vault (encrypt secrets)
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-playbook playbook.yml --ask-vault-pass

# List hosts in inventory
ansible-inventory -i inventory --list
```

### 💡 Lessons Learned
1. **Idempotency is key** — Always use modules instead of raw `command`/`shell` when possible
2. **Use `--check --diff`** to preview changes before running for real
3. **Handlers only run once** — Even if notified multiple times; they run at end of play
4. **Variables precedence** is complex — `defaults < vars < playbook vars < extra vars (-e)`
5. **Ansible Vault** for secrets — Never commit plaintext passwords
6. **Roles for reusability** — Once a playbook grows, refactor into roles
7. **Tags** allow running specific parts: `ansible-playbook playbook.yml --tags "deploy"`
8. **Ansible is not a monitoring tool** — Use it for desired state, not continuous checking

### ❓ Interview Questions
1. **Q: What makes Ansible agentless?**
   A: It connects via SSH (or WinRM for Windows) and requires only Python on the managed host. No daemon or agent to install/maintain.

2. **Q: What is idempotency in Ansible?**
   A: Running a playbook multiple times produces the same result. If a package is already installed, Ansible skips it. This makes playbooks safe to re-run.

3. **Q: Difference between `command` and `shell` modules?**
   A: `command` runs a binary directly (no shell features). `shell` processes through /bin/sh (supports pipes, redirects, variables). Prefer `command` for security.

4. **Q: What are handlers?**
   A: Tasks triggered by `notify`. Only run once at the end of a play if notified. Used for service restarts after config changes.

5. **Q: Ansible vs Terraform — when to use which?**
   A: Terraform for infrastructure provisioning (create VMs, networks, LBs). Ansible for configuration management (install software, configure services on existing servers).

---

## 13. AWS Cloud

### 📖 What Is It?
Amazon Web Services (AWS) is the world's leading cloud platform, offering 200+ services for compute, storage, networking, databases, ML, and more.

### 🎯 Why Is It Used?
- Market leader (~32% market share)
- Most mature and feature-rich cloud
- Global infrastructure (30+ regions)
- Pay-as-you-go pricing
- Vast ecosystem and community

### 🔧 How Is It Used?
Provision infrastructure through Console, CLI, SDKs, or IaC tools (Terraform, CloudFormation). Architect solutions using AWS services.

### 🧩 Key Concepts

#### Core Services (Must Know)

```
COMPUTE:
  EC2           → Virtual machines (instances)
  Lambda        → Serverless functions (event-driven, 15min max)
  ECS           → Container orchestration (Docker on AWS)
  EKS           → Managed Kubernetes
  Fargate       → Serverless containers (no EC2 to manage)

STORAGE:
  S3            → Object storage (files, backups, static hosting)
  EBS           → Block storage (EC2 disk volumes)
  EFS           → Shared file system (NFS, multi-AZ)
  Glacier       → Archival storage (cheap, slow retrieval)

NETWORKING:
  VPC           → Virtual Private Cloud (your own network)
  Subnets       → Public (internet access) / Private (internal only)
  Security Groups → Instance-level firewall (stateful)
  NACLs         → Subnet-level firewall (stateless)
  Route Tables  → Define traffic routing
  IGW           → Internet Gateway (public internet access)
  NAT Gateway   → Private subnet → internet (outbound only)
  ALB/NLB       → Application/Network Load Balancer
  Route 53      → DNS service
  CloudFront    → CDN (content delivery network)
  VPC Peering   → Connect two VPCs
  Transit GW    → Hub for multiple VPC connections

DATABASES:
  RDS           → Managed relational DB (MySQL, PostgreSQL, etc.)
  Aurora        → AWS-optimized MySQL/PostgreSQL (5x performance)
  DynamoDB      → NoSQL key-value store (serverless, fast)
  ElastiCache   → Managed Redis/Memcached
  DocumentDB    → MongoDB-compatible

SECURITY:
  IAM           → Identity & Access Management (users, roles, policies)
  KMS           → Key Management Service (encryption keys)
  Secrets Manager → Store/rotate secrets
  WAF           → Web Application Firewall
  GuardDuty     → Threat detection
  Security Hub  → Security posture overview

MESSAGING & INTEGRATION:
  SQS           → Message queue (decouple services)
  SNS           → Pub/Sub notifications
  EventBridge   → Event bus (serverless event routing)

MONITORING:
  CloudWatch    → Metrics, logs, alarms
  CloudTrail    → API activity audit log
  X-Ray         → Distributed tracing

CI/CD:
  CodePipeline  → CI/CD pipeline service
  CodeBuild     → Build service
  CodeDeploy    → Deployment automation
  ECR           → Elastic Container Registry
```

#### IAM (Identity & Access Management)
```
Principals:
  - Users (humans)
  - Roles (assumed by services/applications)
  - Groups (collection of users)

Policies (JSON):
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}

Best Practices:
  - Least privilege (only permissions needed)
  - Use roles, not long-lived access keys
  - Enable MFA
  - Use IAM policies for access control
  - Service accounts use IAM Roles (not user keys)
```

#### VPC Architecture
```
Region (eu-west-1)
└── VPC (10.0.0.0/16)
    ├── Public Subnet AZ-a (10.0.1.0/24)
    │   ├── NAT Gateway
    │   ├── ALB
    │   └── Bastion Host
    ├── Public Subnet AZ-b (10.0.2.0/24)
    │   └── ALB
    ├── Private Subnet AZ-a (10.0.10.0/24)
    │   └── App Servers (EC2/ECS)
    ├── Private Subnet AZ-b (10.0.20.0/24)
    │   └── App Servers (EC2/ECS)
    ├── Private Subnet AZ-a (10.0.100.0/24)
    │   └── Database (RDS Primary)
    └── Private Subnet AZ-b (10.0.200.0/24)
        └── Database (RDS Standby)

Traffic Flow:
  Internet → IGW → ALB (public subnet) → App (private subnet) → DB (private subnet)
  App → NAT GW → IGW → Internet (for updates, API calls)
```

#### AWS CLI Essentials
```bash
# Configure
aws configure                          # Set up credentials
aws sts get-caller-identity            # Who am I?

# EC2
aws ec2 describe-instances --filters "Name=tag:Environment,Values=prod"
aws ec2 start-instances --instance-ids i-12345
aws ec2 stop-instances --instance-ids i-12345

# S3
aws s3 ls                              # List buckets
aws s3 cp file.txt s3://bucket/        # Upload
aws s3 sync ./dist s3://bucket/        # Sync directory
aws s3 rb s3://bucket --force          # Delete bucket

# ECS
aws ecs list-clusters
aws ecs describe-services --cluster my-cluster --services my-service
aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment

# Logs
aws logs tail /aws/ecs/my-service --follow
aws logs filter-log-events --log-group-name /aws/ecs/my-service --filter-pattern "ERROR"

# SSM (connect without SSH key)
aws ssm start-session --target i-12345
```

### 💡 Lessons Learned
1. **Multi-AZ everything** in production — Single AZ = single point of failure
2. **Use IAM roles, not access keys** — Roles are temporary, auto-rotated
3. **Enable CloudTrail** from day one — You'll need the audit trail eventually
4. **S3 is not a filesystem** — It's object storage; design accordingly
5. **Cost management** — Set up billing alarms, use AWS Cost Explorer, right-size instances
6. **Security groups are additive** — You can't deny; you can only allow. Use NACLs for deny rules
7. **NAT Gateways are expensive** — Consider NAT instances for dev, VPC endpoints for AWS services
8. **Tag everything** — Consistent tagging enables cost tracking, automation, and access control
9. **Use Parameter Store / Secrets Manager** — Never hardcode credentials
10. **Understand the shared responsibility model** — AWS secures the cloud; you secure what's IN the cloud

### ❓ Interview Questions
1. **Q: What is the difference between Security Groups and NACLs?**
   A: SGs are stateful (return traffic auto-allowed), instance-level, allow-only. NACLs are stateless, subnet-level, support allow AND deny rules.

2. **Q: How does a VPC work?**
   A: VPC is your private network in AWS. You define CIDR blocks, create subnets across AZs, attach gateways, and control traffic with route tables and security groups.

3. **Q: What is an IAM Role vs IAM User?**
   A: User has permanent credentials (access key). Role provides temporary credentials via STS, can be assumed by services, applications, or other accounts.

4. **Q: How do you make an application highly available?**
   A: Deploy across multiple AZs, use Auto Scaling Groups, put a Load Balancer in front, use Multi-AZ RDS, and design for failure.

5. **Q: What is the difference between S3 storage classes?**
   A: Standard (frequent access), IA (infrequent, lower cost), Glacier (archive, minutes-hours retrieval), Deep Archive (cheapest, 12-hour retrieval).

---

## 14. Azure Cloud

### 📖 What Is It?
Microsoft Azure is the second-largest cloud platform, strong in enterprise, hybrid cloud, and Microsoft ecosystem integration.

### 🎯 Why Is It Used?
- Seamless integration with Microsoft tools (Active Directory, Office 365)
- Strong hybrid cloud story (Azure Arc, Azure Stack)
- Second-largest market share (~23%)
- Excellent enterprise compliance and governance tools
- Growing Kubernetes support (AKS)

### 🧩 Key Concepts (AWS → Azure Mapping)

```
AWS                  →  Azure
EC2                  →  Virtual Machines
Lambda               →  Azure Functions
ECS/EKS              →  AKS (Azure Kubernetes Service)
S3                   →  Blob Storage
EBS                  →  Managed Disks
VPC                  →  Virtual Network (VNet)
ALB                  →  Application Gateway
Route 53             →  Azure DNS
RDS                  →  Azure SQL / Azure Database
IAM                  →  Azure AD (Entra ID) + RBAC
CloudWatch           →  Azure Monitor
CloudFormation       →  ARM Templates / Bicep
CodePipeline         →  Azure DevOps Pipelines
ECR                  →  Azure Container Registry (ACR)
Secrets Manager      →  Azure Key Vault
SQS/SNS             →  Azure Service Bus / Event Grid
```

### 💡 Lessons Learned
1. **Azure AD (Entra ID) is central** — Everything authenticates through it
2. **Resource Groups** are mandatory — Every resource belongs to one; use them for lifecycle management
3. **Bicep > ARM Templates** — Bicep is cleaner, compiles to ARM, and is the future of Azure IaC
4. **Azure DevOps** is a complete platform (repos, boards, pipelines, artifacts, test plans)
5. **Use Managed Identities** — Azure's equivalent of IAM roles (no credential management)

---

## 15. Monitoring & Observability

### 📖 What Is It?
Monitoring and observability are practices for understanding system health, performance, and behavior. The "three pillars" are Metrics, Logs, and Traces.

### 🎯 Why Is It Used?
- Detect issues before users are affected
- Understand system behavior and performance
- Enable fast root cause analysis
- Capacity planning and optimization
- Meet SLAs and SLOs

### 🔧 How Is It Used?
Instrument applications → Collect metrics/logs/traces → Visualize in dashboards → Set up alerts → Respond to incidents.

### 🧩 Key Concepts

#### Three Pillars of Observability
```
METRICS (Numbers over time):
  - CPU, memory, disk, network usage
  - Request rate, error rate, response time (RED method)
  - Business metrics (orders/min, signups/day)
  - Tools: Prometheus, Datadog, CloudWatch

LOGS (Events with context):
  - Application logs (errors, warnings, info)
  - Access logs (who accessed what)
  - Audit logs (who changed what)
  - Tools: ELK Stack, Loki, CloudWatch Logs

TRACES (Request journey across services):
  - Follow a single request across microservices
  - Identify bottlenecks and failures
  - Tools: Jaeger, Zipkin, X-Ray, Tempo
```

#### Prometheus + Grafana

```yaml
# Prometheus Architecture:
# App → /metrics endpoint → Prometheus scrapes → PromQL queries → Grafana dashboards → Alertmanager → PagerDuty/Slack

# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alerts/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

#### PromQL (Prometheus Query Language)
```promql
# Request rate (per second, 5m window)
rate(http_requests_total[5m])

# Error rate percentage
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Memory usage
container_memory_usage_bytes{namespace="production"}

# Disk usage percentage
(node_filesystem_size_bytes - node_filesystem_avail_bytes) / node_filesystem_size_bytes * 100
```

#### Alert Rules
```yaml
# alerts/app.yml
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate (> 5%)"
          description: "{{ $labels.instance }} has error rate of {{ $value | humanizePercentage }}"

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High p95 latency (> 500ms)"

      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Pod {{ $labels.pod }} is crash looping"
```

#### Golden Signals (Google SRE)
```
1. Latency      → How long requests take (p50, p95, p99)
2. Traffic      → How much demand (requests/sec)
3. Errors       → Rate of failed requests
4. Saturation   → How full the system is (CPU, memory, queue depth)
```

#### RED Method (for services)
```
Rate    → Requests per second
Errors  → Errors per second
Duration → Time per request (latency distribution)
```

#### USE Method (for infrastructure)
```
Utilization → % of resource busy (CPU 80%)
Saturation  → Queue depth, work waiting
Errors      → Error count (disk errors, network drops)
```

### 💡 Lessons Learned
1. **Alert on symptoms, not causes** — Alert on "high error rate" not "CPU is high"
2. **Avoid alert fatigue** — Only alert on actionable items that need human intervention
3. **Dashboard hierarchy** — Overview → Service → Component (drill down)
4. **Label everything consistently** — Environment, team, service name
5. **Retention matters** — Keep detailed metrics for 15 days, aggregated for longer
6. **Instrument from day one** — Adding observability to a running system is 10x harder
7. **Structured logging** (JSON) — Makes searching and parsing possible at scale
8. **Correlation IDs** — Pass a unique ID across services to trace requests

### ❓ Interview Questions
1. **Q: What is the difference between monitoring and observability?**
   A: Monitoring tells you WHEN something is wrong (predefined checks). Observability lets you understand WHY by exploring metrics, logs, and traces — especially for unknown-unknowns.

2. **Q: What are the Golden Signals?**
   A: Latency, Traffic, Errors, Saturation. From Google SRE book. These four metrics cover the health of any service.

3. **Q: How does Prometheus collect metrics?**
   A: Pull-based model. Prometheus scrapes HTTP endpoints (/metrics) at configured intervals. Targets are discovered via static config or service discovery.

4. **Q: What is a Service Level Objective (SLO)?**
   A: A target for service reliability (e.g., "99.9% of requests complete in under 200ms"). Based on Service Level Indicators (SLIs).

---

## 16. Logging (ELK/EFK Stack)

### 📖 What Is It?
Centralized logging solutions that collect, store, and search logs from all your applications and infrastructure in one place.

### 🎯 Why Is It Used?
- Searching logs across hundreds of containers/servers
- Correlating events across services
- Root cause analysis during incidents
- Compliance and auditing
- Pattern detection (repeated errors, anomalies)

### 🧩 Key Concepts

#### ELK Stack
```
Elasticsearch  → Search engine & storage (indexes logs)
Logstash       → Log processing pipeline (parse, transform, route)
Kibana         → Visualization & dashboards (UI for searching)

EFK Stack (Kubernetes):
Elasticsearch  → Search & storage
Fluentd/Fluent Bit → Lightweight log collector (replaces Logstash)
Kibana         → Visualization

Modern Alternative:
Grafana Loki   → Like Prometheus but for logs (label-based, cost-effective)
Promtail       → Log collector for Loki
Grafana        → Visualization
```

#### Structured Logging (Best Practice)
```json
// BAD — unstructured
"User john logged in from 192.168.1.100"

// GOOD — structured (JSON)
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "info",
  "service": "auth-service",
  "event": "user_login",
  "user_id": "john",
  "source_ip": "192.168.1.100",
  "correlation_id": "abc-123-def",
  "duration_ms": 45
}
```

#### Log Levels
```
FATAL   → System is unusable, immediate action needed
ERROR   → Something failed, needs attention
WARN    → Potential issue, system still works
INFO    → Normal operations (user login, request served)
DEBUG   → Detailed for troubleshooting (not in production)
TRACE   → Very detailed (rarely used, performance hit)

Production: INFO and above
Debugging: DEBUG temporarily, revert after
```

### 💡 Lessons Learned
1. **Structured logs (JSON) are mandatory** for searchability at scale
2. **Include correlation IDs** in every log line — enables request tracing
3. **Don't log sensitive data** — PII, passwords, tokens, credit cards
4. **Log rotation and retention policies** — Logs grow fast; archive or delete old ones
5. **Separate log streams** — Access logs, application logs, error logs → different indexes/retention
6. **Loki is much cheaper than Elasticsearch** — Consider it for K8s environments

---

## 17. Helm (Kubernetes Package Manager)

### 📖 What Is It?
Helm is a package manager for Kubernetes. It packages related K8s manifests into "charts" that can be versioned, shared, and deployed with custom values.

### 🎯 Why Is It Used?
- Template Kubernetes manifests (avoid duplication)
- Manage complex multi-resource applications as one unit
- Version and rollback deployments
- Share reusable packages (public charts for Redis, PostgreSQL, etc.)
- Environment-specific configuration via values files

### 🔧 How Is It Used?
Create or use existing charts → Override values per environment → `helm install/upgrade` → Kubernetes resources created.

### 🧩 Key Concepts

#### Chart Structure
```
mychart/
├── Chart.yaml        # Chart metadata (name, version)
├── values.yaml       # Default configuration values
├── templates/        # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── hpa.yaml
│   ├── _helpers.tpl  # Template helpers
│   └── NOTES.txt     # Post-install instructions
├── charts/           # Sub-charts (dependencies)
└── .helmignore
```

#### Helm Commands
```bash
# Repository management
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo redis

# Install / Upgrade
helm install myrelease ./mychart -n namespace
helm install redis bitnami/redis -f values-prod.yaml
helm upgrade myrelease ./mychart -f values.yaml --install  # Install or upgrade
helm upgrade --set image.tag=2.0.0 myrelease ./mychart

# Information
helm list -n namespace            # List releases
helm status myrelease             # Release status
helm get values myrelease         # Current values
helm get manifest myrelease       # Rendered manifests
helm history myrelease            # Release history

# Rollback
helm rollback myrelease 1         # Rollback to revision 1

# Debug
helm template mychart ./mychart -f values.yaml  # Render locally (dry-run)
helm install --debug --dry-run myrelease ./mychart

# Cleanup
helm uninstall myrelease -n namespace
```

#### Template Example
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- if .Values.healthCheck.enabled }}
          livenessProbe:
            httpGet:
              path: {{ .Values.healthCheck.path }}
              port: {{ .Values.service.targetPort }}
          {{- end }}

# values.yaml
replicaCount: 3
image:
  repository: myapp
  tag: "1.0.0"
service:
  type: ClusterIP
  port: 80
  targetPort: 8080
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
healthCheck:
  enabled: true
  path: /health
```

### 💡 Lessons Learned
1. **Use `helm template` to debug** — See rendered manifests before deploying
2. **Pin chart versions** — `helm install redis bitnami/redis --version 17.0.0`
3. **One values file per environment** — `values-dev.yaml`, `values-prod.yaml`
4. **Use `--atomic`** flag — Auto-rollback on failed upgrade
5. **Helm secrets** plugin for encrypted values files (gitops-compatible)
6. **Don't over-template** — If it's the same everywhere, hard-code it

---

## 18. Service Mesh (Istio)

### 📖 What Is It?
A service mesh is a dedicated infrastructure layer that handles service-to-service communication. It provides traffic management, security, and observability without changing application code.

### 🎯 Why Is It Used?
- Mutual TLS (mTLS) between all services — zero-trust networking
- Traffic management (canary, A/B testing, fault injection)
- Observability (automatic metrics, traces for all traffic)
- Retry, timeout, and circuit-breaking policies
- Fine-grained access control between services

### 🧩 Key Concepts
```
Architecture:
  Data Plane   → Sidecar proxies (Envoy) injected into every pod
  Control Plane → Manages configuration (Istiod)

Key Features:
  Traffic Management:
    - VirtualService    → Routing rules (canary, header-based)
    - DestinationRule   → Load balancing, circuit breaking
    - Gateway           → Ingress/egress traffic management

  Security:
    - mTLS everywhere (automatic certificate rotation)
    - AuthorizationPolicy (which service can call which)
    - PeerAuthentication (enforce mTLS mode)

  Observability:
    - Automatic metrics for all traffic (no code changes)
    - Distributed tracing (Jaeger integration)
    - Service graph visualization (Kiali)
```

#### Canary Deployment with Istio
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
    - myapp
  http:
    - route:
        - destination:
            host: myapp
            subset: v1
          weight: 90
        - destination:
            host: myapp
            subset: v2
          weight: 10  # 10% to canary
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

### 💡 Lessons Learned
1. **Service mesh adds complexity** — Don't adopt it until you have 10+ services
2. **Sidecar proxies use resources** — Plan for extra CPU/memory per pod
3. **Start with mTLS only** — Get security benefits before adding traffic management
4. **Istio ambient mesh** (newer) — No sidecars, uses node-level proxies (simpler)
5. **Alternatives**: Linkerd (simpler, lighter), Consul Connect (HashiCorp)

---

## 19. Security & DevSecOps

### 📖 What Is It?
DevSecOps integrates security practices into every phase of the DevOps lifecycle. Security is everyone's responsibility, not just the security team's.

### 🎯 Why Is It Used?
- Shift security left (find issues early, cheaper to fix)
- Automated security scanning in CI/CD
- Comply with regulations (SOC2, GDPR, HIPAA, PCI-DSS)
- Reduce attack surface
- Build security into culture, not bolt it on

### 🧩 Key Concepts

#### Security in the Pipeline
```
Code Phase:
  - Pre-commit hooks (secrets detection)
  - IDE security plugins
  - Peer review (code review)

Build Phase:
  - SAST (Static Application Security Testing) → Scan source code
    Tools: SonarQube, Semgrep, CodeQL
  - Dependency scanning → Check for vulnerable libraries
    Tools: Snyk, Dependabot, OWASP Dependency-Check
  - License compliance

Test Phase:
  - DAST (Dynamic Application Security Testing) → Scan running app
    Tools: OWASP ZAP, Burp Suite
  - Container image scanning
    Tools: Trivy, Grype, Docker Scout

Deploy Phase:
  - Infrastructure scanning (Terraform)
    Tools: Checkov, tfsec, KICS
  - Kubernetes security
    Tools: kube-bench, Falco, OPA/Gatekeeper
  - Secret management
    Tools: Vault, AWS Secrets Manager, Sealed Secrets

Runtime:
  - Network policies (zero-trust)
  - Runtime threat detection (Falco)
  - Audit logging
  - Automated patching
```

#### Container Security Checklist
```
□ Use minimal base images (distroless, alpine)
□ Run as non-root user
□ Scan images for CVEs (Trivy)
□ Sign images (cosign/Notary)
□ Use read-only filesystem where possible
□ Don't store secrets in images
□ Pin base image versions (avoid :latest)
□ Limit container capabilities (drop ALL, add specific)
□ Set resource limits (prevent DoS)
□ Use NetworkPolicies to restrict traffic
```

#### Kubernetes Security
```yaml
# Pod Security — Best Practice
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
      resources:
        limits:
          memory: "256Mi"
          cpu: "500m"
```

#### Secret Management
```
NEVER:
  - Hardcode secrets in code
  - Store secrets in environment variables in Dockerfiles
  - Commit secrets to git
  - Use default passwords

ALWAYS:
  - Use a secrets manager (Vault, AWS Secrets Manager, Azure Key Vault)
  - Rotate secrets regularly
  - Use short-lived credentials (IAM roles, service accounts)
  - Encrypt secrets at rest and in transit
  - Audit secret access

For Kubernetes:
  - External Secrets Operator (sync from Vault/AWS to K8s secrets)
  - Sealed Secrets (encrypted in git, decrypted in cluster)
  - CSI Secret Store Driver
```

#### OWASP Top 10 (Know These)
```
1. Broken Access Control
2. Cryptographic Failures
3. Injection (SQL, NoSQL, Command)
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable Components
7. Authentication Failures
8. Data Integrity Failures
9. Logging & Monitoring Failures
10. Server-Side Request Forgery (SSRF)
```

### 💡 Lessons Learned
1. **Automate security scanning** — If it's manual, it won't happen consistently
2. **Treat security findings like bugs** — Track, prioritize, fix
3. **Zero-trust networking** — Never assume internal traffic is safe
4. **Least privilege everywhere** — IAM, K8s RBAC, file permissions, network policies
5. **Rotate secrets automatically** — Manual rotation means stale credentials
6. **If a secret is exposed, rotate immediately** — Don't just delete from code
7. **Security is a spectrum** — Start somewhere, improve continuously
8. **Immutable infrastructure** — Don't patch running systems; rebuild and redeploy

### ❓ Interview Questions
1. **Q: What is shift-left security?**
   A: Moving security practices earlier in the development lifecycle (coding, building) instead of only testing at the end. Catches issues when they're cheaper to fix.

2. **Q: How do you manage secrets in Kubernetes?**
   A: Use External Secrets Operator or Sealed Secrets instead of plain K8s Secrets. Sync from Vault or cloud secret managers. Never commit plain secrets to git.

3. **Q: What is the difference between SAST and DAST?**
   A: SAST scans source code without running it (finds code-level vulnerabilities). DAST tests the running application from outside (finds runtime vulnerabilities like XSS, auth issues).

4. **Q: What is a supply chain attack?**
   A: Compromising a dependency, build tool, or CI/CD pipeline to inject malicious code. Mitigations: pin versions, verify signatures, use private registries, scan dependencies.

---

## 20. Python for DevOps

### 📖 What Is It?
Python is the go-to scripting language for DevOps automation beyond what Bash can handle. It's used for building tools, API integrations, cloud automation, and data processing.

### 🎯 Why Is It Used?
- Readable and concise syntax
- Rich standard library + vast ecosystem (boto3, requests, paramiko)
- Cloud SDK support (AWS boto3, Azure SDK, GCP client)
- Great for API automation and webhook handlers
- Ansible, many DevOps tools written in Python

### 🧩 Key Concepts

#### Essential for DevOps
```python
# === Working with APIs ===
import requests

response = requests.get("https://api.github.com/repos/owner/repo",
                       headers={"Authorization": f"Bearer {token}"})
if response.status_code == 200:
    data = response.json()
    print(f"Stars: {data['stargazers_count']}")

# === AWS Automation (boto3) ===
import boto3

ec2 = boto3.client('ec2', region_name='eu-west-1')

# Stop all dev instances
instances = ec2.describe_instances(
    Filters=[{'Name': 'tag:Environment', 'Values': ['dev']}]
)
for reservation in instances['Reservations']:
    for instance in reservation['Instances']:
        ec2.stop_instances(InstanceIds=[instance['InstanceId']])

# === File Operations ===
import json, yaml, os
from pathlib import Path

# Read YAML config
with open('config.yaml') as f:
    config = yaml.safe_load(f)

# Read/write JSON
with open('data.json') as f:
    data = json.load(f)

# Environment variables
db_host = os.environ.get('DB_HOST', 'localhost')

# === Subprocess (running commands) ===
import subprocess

result = subprocess.run(
    ['kubectl', 'get', 'pods', '-n', 'production', '-o', 'json'],
    capture_output=True, text=True, check=True
)
pods = json.loads(result.stdout)

# === Error Handling ===
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

try:
    response = requests.post(url, json=payload, timeout=30)
    response.raise_for_status()
except requests.exceptions.Timeout:
    logger.error("Request timed out")
except requests.exceptions.HTTPError as e:
    logger.error(f"HTTP error: {e.response.status_code}")
```

#### Useful Libraries for DevOps
```
boto3        → AWS SDK
requests     → HTTP client
paramiko     → SSH client
pyyaml       → YAML parsing
jinja2       → Templating
click/typer  → CLI tools
docker       → Docker SDK
kubernetes   → Kubernetes client
fabric       → Remote execution
schedule     → Task scheduling
```

### 💡 Lessons Learned
1. **Use virtual environments** (`venv` or `poetry`) — Don't pollute system Python
2. **Pin dependencies** in `requirements.txt` with exact versions
3. **Use `try/except` for external calls** — Network, APIs, and disk can all fail
4. **Logging > print** — Use the logging module for production scripts
5. **Type hints** improve readability: `def deploy(service: str, version: str) -> bool:`

---

## 21. Site Reliability Engineering (SRE)

### 📖 What Is It?
SRE is Google's approach to operations. It applies software engineering practices to infrastructure and operations problems. "SRE is what happens when a software engineer is asked to design an operations team."

### 🎯 Why Is It Used?
- Balance reliability with feature velocity
- Define measurable reliability targets (SLOs)
- Error budgets allow calculated risk-taking
- Reduce toil through automation
- Blameless incident management

### 🧩 Key Concepts

#### SLI, SLO, SLA
```
SLI (Service Level Indicator):
  - A metric that measures service reliability
  - Example: "99.2% of requests complete in < 200ms"
  - Example: "99.95% availability this month"

SLO (Service Level Objective):
  - A target value for an SLI
  - Internal goal (e.g., 99.9% availability)
  - Set by the team based on user expectations

SLA (Service Level Agreement):
  - A contract with customers
  - Has consequences if broken (refunds, penalties)
  - Should be less strict than SLO (buffer)

Example:
  SLI: "Percentage of requests returning 2xx in < 500ms"
  SLO: "99.9% of requests meet the SLI (monthly)"
  SLA: "99.5% uptime guaranteed to customers"
```

#### Error Budget
```
If SLO = 99.9% availability per month:
  30 days × 24 hours × 60 minutes = 43,200 minutes
  Error budget = 0.1% = 43.2 minutes of downtime allowed

How to use:
  - Budget remaining → ship features, take risks
  - Budget depleted → freeze deployments, focus on reliability
  - Error budget policy defines actions at different thresholds
```

#### Toil
```
Toil = Manual, repetitive, automatable operational work

Characteristics:
  - Manual (human runs it)
  - Repetitive (happens regularly)
  - Automatable (could be scripted)
  - No lasting value (doesn't improve the system)
  - Scales with service growth

Goal: Keep toil < 50% of an SRE's time
Examples of toil:
  - Manually restarting services
  - Manually scaling infrastructure
  - Repetitive ticket responses
  - Manual deployments
```

#### Incident Management
```
1. Detect    → Monitoring alerts fire
2. Respond   → On-call acknowledges, starts investigation
3. Mitigate  → Stop the bleeding (rollback, scale up, failover)
4. Resolve   → Fix root cause
5. Postmortem → Blameless review, action items

Postmortem Template:
  - Incident summary
  - Timeline of events
  - Root cause analysis (5 Whys)
  - Impact (duration, users affected, revenue lost)
  - What went well
  - What went poorly
  - Action items (with owners and deadlines)
```

### 💡 Lessons Learned
1. **SLOs drive decisions** — They tell you when to invest in reliability vs features
2. **100% is the wrong target** — It's impossible and too expensive; users don't notice 99.99% vs 100%
3. **Automate yourself out of toil** — If you do it more than twice, automate it
4. **Blameless postmortems** — Blame prevents learning; focus on systems, not individuals
5. **On-call should be sustainable** — Good documentation, runbooks, and escalation paths
6. **Runbooks are critical** — Document common incidents and their resolution steps

### ❓ Interview Questions
1. **Q: What is an error budget?**
   A: The allowed amount of unreliability. If SLO is 99.9%, you have 0.1% error budget. While budget remains, you can ship features. When depleted, focus shifts to reliability.

2. **Q: What is toil and how do you reduce it?**
   A: Toil is manual, repetitive operational work. Reduce by automating (scripts, self-healing systems), improving tools, and eliminating root causes of repetitive incidents.

3. **Q: Describe a blameless postmortem.**
   A: A review focused on what happened and why, not who did it. Documents timeline, root cause, impact, and action items. Assumes good intent. Goal is systemic improvement.

---

## 22. ArgoCD & GitOps

### 📖 What Is It?
GitOps is a pattern where Git is the single source of truth for infrastructure and application configuration. ArgoCD is a Kubernetes-native GitOps tool that syncs cluster state with a Git repository.

### 🎯 Why Is It Used?
- Git as single source of truth (auditable, versioned)
- Automatic drift detection and correction
- Declarative deployments (desired state in Git)
- Easy rollbacks (revert Git commit = rollback deployment)
- Self-healing (ArgoCD reconciles drift automatically)

### 🔧 How Is It Used?
Define K8s manifests in Git → ArgoCD watches the repo → Detects differences → Syncs cluster to match Git state.

### 🧩 Key Concepts

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/k8s-manifests.git
    targetRevision: main
    path: apps/myapp/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true        # Delete resources removed from Git
      selfHeal: true     # Revert manual changes
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 3m
```

#### GitOps Repository Structure
```
k8s-manifests/
├── apps/
│   ├── myapp/
│   │   ├── base/                    # Common manifests
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       ├── dev/                 # Dev overrides
│   │       │   ├── kustomization.yaml
│   │       │   └── replica-patch.yaml
│   │       ├── staging/
│   │       └── production/
│   └── another-app/
└── infrastructure/
    ├── monitoring/
    ├── ingress/
    └── cert-manager/
```

#### GitOps Workflow
```
Developer pushes code → CI builds image → CI updates image tag in manifest repo → 
ArgoCD detects change → ArgoCD syncs to cluster → Application updated

Key Principle:
  CI Pipeline: Build & Test → Push artifact → Update manifest
  CD (ArgoCD): Watch manifest repo → Sync to cluster

Separation:
  - Application repo (source code + Dockerfile)
  - Manifest repo (K8s YAML + Helm values)
```

### 💡 Lessons Learned
1. **Separate app repo and config repo** — CI updates config repo; ArgoCD syncs it
2. **Use Kustomize or Helm** for environment-specific overlays
3. **Enable self-heal** — Prevents configuration drift from manual kubectl changes
4. **Image Updater** — ArgoCD Image Updater can auto-update image tags
5. **RBAC in ArgoCD** — Restrict who can sync to production
6. **Sealed Secrets or External Secrets** for GitOps-compatible secret management

---

## 23. Databases for DevOps

### 📖 What Is It?
Understanding databases from an operational perspective: provisioning, scaling, backup/restore, monitoring, and troubleshooting.

### 🎯 Why Is It Used?
- Every application needs a data store
- Database issues cause the most impactful outages
- DevOps owns database infrastructure (provisioning, scaling, backups)
- Performance tuning often falls on DevOps/SRE

### 🧩 Key Concepts

#### SQL vs NoSQL
```
SQL (Relational):
  - Structured data with relationships
  - ACID transactions (Atomicity, Consistency, Isolation, Durability)
  - Fixed schema
  - Examples: PostgreSQL, MySQL, SQL Server
  - Use when: Data integrity critical, complex queries, relationships

NoSQL:
  - Document: MongoDB, CouchDB (JSON-like documents)
  - Key-Value: Redis, DynamoDB (fast lookups)
  - Column: Cassandra (time-series, IoT)
  - Graph: Neo4j (relationships/connections)
  - Use when: Flexible schema, massive scale, specific access patterns
```

#### Database Operations
```bash
# PostgreSQL
pg_dump dbname > backup.sql              # Backup
psql dbname < backup.sql                 # Restore
pg_isready -h localhost -p 5432          # Health check
SELECT * FROM pg_stat_activity;          # Active connections
SELECT pg_size_pretty(pg_database_size('dbname'));  # DB size

# MySQL
mysqldump -u root -p dbname > backup.sql
mysql -u root -p dbname < backup.sql
SHOW PROCESSLIST;                        # Active queries
SHOW TABLE STATUS;                       # Table sizes
```

#### Key Operational Concerns
```
Backups:
  - Automated daily backups (retention 7-30 days)
  - Point-in-time recovery (WAL/binlog)
  - Test restores regularly!
  - Store backups in separate region/account

High Availability:
  - Primary-Replica (read replicas)
  - Multi-AZ (automatic failover)
  - Connection pooling (PgBouncer, ProxySQL)

Monitoring:
  - Connections (current vs max)
  - Query latency (slow query log)
  - Replication lag
  - Disk usage and growth rate
  - Lock contention
  - Cache hit ratio
```

### 💡 Lessons Learned
1. **Test your backups by restoring them** — Untested backups aren't backups
2. **Connection pooling is essential** — Apps creating direct connections will exhaust DB limits
3. **Never run migrations during peak hours** — Schedule during low-traffic windows
4. **Managed databases save toil** — RDS, Cloud SQL handle patching, backups, failover
5. **Index wisely** — Missing indexes = slow queries; too many indexes = slow writes
6. **Replication lag is normal** — Design read-after-write patterns accordingly
7. **Database passwords rotate** — Use secrets managers with automatic rotation

---

## 24. Soft Skills & Career Growth

### 📖 What Is It?
Technical skills get you hired; soft skills get you promoted. Communication, collaboration, and operational practices are what make a DevOps engineer effective.

### 🧩 Key Concepts

#### Communication
```
Documentation:
  - Write runbooks for every service you own
  - Document architecture decisions (ADRs)
  - Keep READMEs up to date
  - Write postmortems after incidents

Communication Patterns:
  - Be clear and concise in Slack/email
  - Over-communicate during incidents (status updates)
  - Say "I don't know, but I'll find out" — it's respected
  - Write for your audience (developer vs manager vs executive)
```

#### Incident Response
```
On-Call Best Practices:
  1. Acknowledge alerts within 5 minutes
  2. Assess severity (P1-P4)
  3. Communicate early ("Investigating issue with X")
  4. Mitigate first, root-cause later
  5. Update stakeholders every 15-30 minutes
  6. Escalate when stuck (no shame in escalation)
  7. Write postmortem within 48 hours

Severity Levels:
  P1: Complete service outage (all hands)
  P2: Major degradation (immediate response)
  P3: Minor impact (business hours fix)
  P4: Cosmetic/low impact (backlog)
```

#### Career Growth Path
```
Junior DevOps:
  - Linux, Git, basic CI/CD
  - Docker basics
  - Cloud fundamentals (one provider)
  - Scripting (Bash, Python basics)

Mid-Level DevOps:
  - Kubernetes administration
  - Terraform/IaC proficiency
  - Multiple CI/CD platforms
  - Monitoring & observability
  - Security basics

Senior DevOps/SRE:
  - Architecture design
  - Platform engineering
  - Cost optimization
  - Mentoring others
  - Incident management leadership
  - Multi-cloud
  - Service mesh, GitOps

Staff/Principal:
  - Technical strategy
  - Organizational influence
  - Building internal platforms
  - Setting standards and best practices
  - Cross-team collaboration
```

#### Interview Tips
```
Behavioral Questions (STAR Method):
  Situation → Task → Action → Result

Common DevOps Interview Scenarios:
  - "Tell me about a production incident you handled"
  - "How would you design a CI/CD pipeline for..."
  - "Describe your experience with IaC"
  - "How do you handle on-call?"

Technical Interviews:
  - Whiteboard architecture (VPC, K8s cluster, CI/CD flow)
  - Live troubleshooting (given a failing system, debug it)
  - Coding challenge (automation script)
  - Take-home (build a pipeline, deploy an app)

Red Flags to Mention in Interviews:
  - "We deploy manually"
  - "We don't have monitoring"
  - "Our infrastructure isn't in code"
  - "We don't do code reviews"
```

### 💡 Final Wisdom
```
1. You don't need to memorize everything — know WHERE to find it
2. Build small projects to learn (home lab, side projects)
3. Contribute to open source for visibility and learning
4. Teach others — it solidifies your own understanding
5. Stay curious — the field evolves; embrace continuous learning
6. Automate yourself out of repetitive work
7. Understand the business impact of your work
8. Network with other DevOps engineers (meetups, conferences, online)
9. Document everything you learn (like this Second Brain!)
10. Done is better than perfect — ship, iterate, improve
```

---

## 📚 Recommended Resources

### Books
- "The Phoenix Project" — DevOps novel (start here)
- "The DevOps Handbook" — Practical DevOps
- "Site Reliability Engineering" — Google's SRE book (free online)
- "Kubernetes Up & Running" — K8s deep dive
- "Terraform Up & Running" — IaC mastery
- "The Practice of Cloud System Administration" — Distributed systems

### Certifications (in order)
1. AWS Solutions Architect Associate
2. Certified Kubernetes Administrator (CKA)
3. HashiCorp Terraform Associate
4. AWS DevOps Engineer Professional
5. Certified Kubernetes Security Specialist (CKS)

### Practice Platforms
- KodeKloud — Labs for K8s, Docker, Terraform, Ansible
- A Cloud Guru — Cloud certifications
- Killer.sh — CKA/CKS exam practice
- labs.play-with-docker.com — Free Docker playground
- katacoda.com — Interactive scenarios

---

> **🧠 Remember:** This document is a living reference. Update it as you learn new things, encounter new patterns, or discover better approaches. The best Second Brain grows with you.

> **📅 Review Schedule:** Read one section per day. By the end of the month, you'll have reviewed everything. Repetition is the key to retention.
