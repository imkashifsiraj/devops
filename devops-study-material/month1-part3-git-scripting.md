# Month 1 — PART 3: GIT & VERSION CONTROL

---

## 3.1 What is Git?

Git is a distributed version control system. It tracks every change to your code, enables collaboration, and is the foundation of all CI/CD pipelines.

**Why DevOps Engineers need Git:**
- All infrastructure code lives in Git (Terraform, Ansible, K8s manifests)
- CI/CD pipelines trigger from Git events (push, merge, tag)
- GitOps uses Git as the single source of truth
- Code reviews happen through Git (merge/pull requests)
- Rollback = revert a Git commit

---

## 3.2 Git Architecture

```
┌─────────────┐     git add     ┌──────────────┐    git commit    ┌─────────────┐    git push    ┌─────────────┐
│  Working    │ ──────────────> │   Staging    │ ──────────────> │    Local    │ ────────────> │   Remote    │
│  Directory  │                 │ Area (Index) │                  │    Repo     │               │    Repo     │
└─────────────┘                 └──────────────┘                  └─────────────┘               └─────────────┘
       ^                                                                                              │
       │                                                                                              │
       └──────────────────────────── git pull / git fetch + git merge ────────────────────────────────┘

Key concepts:
- Working Directory: Your actual files on disk
- Staging Area: Changes marked for next commit (like a shopping cart)
- Local Repository: Your complete history (.git directory)
- Remote Repository: Shared repo (GitHub, GitLab, Bitbucket)
```

---

## 3.3 Essential Git Commands

### First-Time Setup
```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
git config --global core.editor "vim"           # or "code --wait" for VS Code
git config --global pull.rebase false           # Merge on pull (default)
git config --list                               # Verify settings
```

### Daily Workflow
```bash
# Start a new repo
git init                              # Initialize in current directory
git clone URL                         # Clone existing remote repo
git clone URL directory-name          # Clone into specific directory

# Check status
git status                            # What's changed? (RUN THIS OFTEN)
git status -s                         # Short format

# Stage changes
git add file.txt                      # Stage specific file
git add *.py                          # Stage all Python files
git add .                             # Stage everything (be careful!)
git add -p                            # Interactive staging (patch mode) — GREAT for partial commits

# Commit
git commit -m "Add user authentication feature"
git commit -am "Fix typo in config"   # Add tracked files + commit (shortcut)

# View changes
git diff                              # Unstaged changes
git diff --staged                     # Staged changes (about to commit)
git diff main..feature-branch         # Difference between branches
git diff HEAD~3                       # Changes in last 3 commits

# Push & Pull
git push origin main                  # Push to remote
git push -u origin feature-branch     # Push new branch + set tracking
git pull origin main                  # Fetch + merge from remote
git fetch origin                      # Download without merging (safer)
```

### Branching (You'll Do This Constantly)
```bash
# Create & switch branches
git branch                            # List local branches
git branch -a                         # List all branches (including remote)
git branch feature-login              # Create branch
git checkout feature-login            # Switch to branch
git checkout -b feature-login         # Create + switch (shortcut)
git switch -c feature-login           # Modern way to create + switch

# Merge
git checkout main                     # Switch to target branch
git merge feature-login               # Merge feature into main
git merge --no-ff feature-login       # Create merge commit (even if fast-forward possible)

# Delete branches
git branch -d feature-login           # Delete (safe — only if merged)
git branch -D feature-login           # Force delete
git push origin --delete feature-login # Delete remote branch

# Track remote branches
git checkout --track origin/feature   # Create local branch tracking remote
git branch -vv                        # Show tracking relationships
```

### Viewing History
```bash
git log                               # Full history
git log --oneline                     # Compact (one line per commit)
git log --oneline --graph --all       # Visual branch graph (BEAUTIFUL)
git log -5                            # Last 5 commits
git log --author="name"               # Commits by author
git log --since="2024-01-01"          # Commits since date
git log -- file.txt                   # History of specific file
git log -p file.txt                   # History with diffs
git show abc1234                      # Show specific commit details
git blame file.txt                    # Who changed each line (and when)
git shortlog -sn                      # Commit count per author
```

### Undoing Things
```bash
# Undo unstaged changes
git checkout -- file.txt              # Discard changes to file
git restore file.txt                  # Modern way to discard changes

# Unstage
git reset HEAD file.txt               # Unstage a file (keep changes)
git restore --staged file.txt         # Modern way to unstage

# Undo commits
git reset --soft HEAD~1               # Undo last commit, keep changes staged
git reset --mixed HEAD~1              # Undo last commit, keep changes unstaged
git reset --hard HEAD~1               # Undo last commit, DISCARD changes (DANGEROUS)

# Safe undo (creates new commit that reverses changes)
git revert abc1234                    # Revert specific commit
git revert HEAD                       # Revert last commit

# Fix last commit message
git commit --amend -m "Better message"

# Add forgotten file to last commit
git add forgotten-file.txt
git commit --amend --no-edit
```

### Stashing (Temporary Shelf)
```bash
git stash                             # Stash current changes
git stash save "work in progress"     # Stash with description
git stash list                        # List all stashes
git stash pop                         # Apply + remove most recent stash
git stash apply                       # Apply but keep in stash list
git stash apply stash@{2}             # Apply specific stash
git stash drop stash@{0}              # Delete specific stash
git stash clear                       # Delete all stashes
git stash show -p                     # Show stash contents (diff)
```

### Tags (Release Markers)
```bash
git tag v1.0.0                        # Lightweight tag
git tag -a v1.0.0 -m "First release"  # Annotated tag (recommended)
git tag                               # List tags
git push origin v1.0.0                # Push specific tag
git push origin --tags                # Push all tags
git checkout v1.0.0                   # Checkout tag (detached HEAD)
```

---

## 3.4 Merge vs Rebase

```
MERGE (preserves history, creates merge commit):

  main:     A---B---C---M
                 \     /
  feature:        D---E

  - Non-destructive
  - Complete history preserved
  - Creates "merge commit" (M)
  - Use for: merging feature branches into main


REBASE (linear history, rewrites commits):

  Before rebase:
  main:     A---B---C
                 \
  feature:        D---E

  After: git checkout feature && git rebase main
  main:     A---B---C
                     \
  feature:            D'---E'  (new commit hashes!)

  - Linear, clean history
  - REWRITES history (new commit hashes)
  - Never rebase public/shared branches!
  - Use for: cleaning up local feature branch before merge


INTERACTIVE REBASE (git rebase -i HEAD~3):
  pick abc1234 Add login page
  squash def5678 Fix typo in login
  squash ghi9012 Another fix

  Combines 3 commits into 1 clean commit
  Use before merge request to keep history clean
```

**Golden Rules:**
1. Never rebase commits that are already pushed/shared
2. Use merge for bringing main into your branch (`git merge main`)
3. Use rebase for cleaning your own local commits before pushing
4. When in doubt, use merge — it's safer

---

## 3.5 Branching Strategies

### Git Flow (Traditional, Enterprise)
```
main        ─────────────────────────────── Only releases
  └── develop ───────────────────────────── Integration
       ├── feature/login ────── ─────────── New features
       ├── feature/dashboard
       ├── release/1.0 ─────────────────── Release prep
       └── hotfix/critical-bug ──────────── Emergency fixes

Pros: Clear structure, release management
Cons: Complex, slow for CI/CD
Best for: Scheduled releases, large teams
```

### Trunk-Based Development (Modern, CI/CD)
```
main  ─────────────────────────────────── Always deployable
  ├── short-lived-branch (hours, max 1-2 days)
  ├── short-lived-branch
  └── short-lived-branch

Pros: Fast, continuous integration, less merge conflicts
Cons: Requires strong CI/CD and feature flags
Best for: DevOps teams, frequent deployments
```

### GitHub Flow (Simple, Popular)
```
main  ─────────────────────────────────── Always deployable
  ├── feature-branch → Pull Request → Merge → Deploy
  ├── feature-branch → Pull Request → Merge → Deploy
  └── feature-branch → Pull Request → Merge → Deploy

Pros: Simple, works with CI/CD
Cons: Less structure for complex releases
Best for: Most teams, web applications
```

---

## 3.6 .gitignore

Tells Git which files to NOT track.

```gitignore
# Dependencies
node_modules/
vendor/
.venv/
__pycache__/

# Build output
dist/
build/
*.pyc
*.class
*.jar

# Environment & Secrets (CRITICAL!)
.env
.env.local
*.pem
*.key
credentials.json
terraform.tfvars

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Terraform
.terraform/
*.tfstate
*.tfstate.backup

# Docker
docker-compose.override.yml

# Logs
*.log
logs/
```

**CRITICAL:** If you accidentally committed a secret:
```bash
# It's NOT enough to just delete the file — it's in history!
# Option 1: BFG Repo Cleaner (easiest)
# Option 2: git filter-branch (complex)
# Option 3: Consider the secret COMPROMISED → rotate it immediately
```

---

## 3.7 Git Best Practices

### Commit Message Convention
```
Format: <type>(<scope>): <subject>

Types:
  feat:     New feature
  fix:      Bug fix
  docs:     Documentation
  refactor: Code refactoring
  test:     Adding tests
  chore:    Maintenance, dependencies
  ci:       CI/CD changes
  perf:     Performance improvement

Examples:
  feat(auth): add JWT token refresh mechanism
  fix(api): handle null response from payment gateway
  docs(readme): add deployment instructions
  ci(gitlab): add container scanning stage
  chore(deps): update terraform provider to v5.1
```

### Pull/Merge Request Best Practices
```
Title: Clear, concise description (< 72 chars)
Body:
  ## What does this MR do?
  Brief description of changes

  ## Why?
  Context/motivation

  ## How to test?
  Steps to verify

  ## Checklist:
  - [ ] Tests passing
  - [ ] Documentation updated
  - [ ] No secrets committed
  - [ ] Reviewed by peer
```

---

## 3.8 Git Free Resources

| Resource | Link | Notes |
|----------|------|-------|
| Learn Git Branching | https://learngitbranching.js.org | Interactive visual tutorial (START HERE) |
| Pro Git Book | https://git-scm.com/book/en/v2 | Free, comprehensive |
| Oh Shit, Git!? | https://ohshitgit.com | Fix common mistakes |
| Git Cheat Sheet | https://education.github.com/git-cheat-sheet-education.pdf | Quick reference |
| Atlassian Git Tutorials | https://www.atlassian.com/git/tutorials | Well-explained guides |
| GitLab Flow | https://docs.gitlab.com/ee/topics/gitlab_flow.html | GitLab's workflow guide |
| Conventional Commits | https://www.conventionalcommits.org | Commit message standard |

---

# PART 4: SHELL SCRIPTING & AUTOMATION

---

## 4.1 What is Shell Scripting?

Shell scripting is writing programs for the Bash shell to automate tasks. It's your first automation tool — before reaching for Python, Ansible, or Terraform.

**When to use Bash vs Python:**
- Bash: Simple automation, file operations, piping commands, < 100 lines
- Python: Complex logic, API interactions, data processing, > 100 lines, error handling

---

## 4.2 Script Basics

```bash
#!/bin/bash
# ^ Shebang — tells the system which interpreter to use
# ALWAYS include this as the first line

# Safety settings (ALWAYS USE THESE in production scripts)
set -euo pipefail
# -e: Exit immediately if any command fails
# -u: Exit if undefined variable is used
# -o pipefail: Pipeline fails if ANY command in pipe fails

# Without set -e:
rm /nonexistent/file    # Fails silently
echo "This still runs"  # This executes (BAD!)

# With set -e:
rm /nonexistent/file    # Fails
echo "This never runs"  # Script exited (GOOD!)
```

### Making Scripts Executable
```bash
chmod +x script.sh      # Make executable
./script.sh             # Run it
bash script.sh          # Or run explicitly with bash
```

---

## 4.3 Variables

```bash
# Assignment (NO spaces around =)
NAME="DevOps"
PORT=8080
ENVIRONMENT="production"

# Usage
echo "Hello, $NAME"
echo "Running on port ${PORT}"      # Braces for clarity

# Command substitution
TODAY=$(date +%Y-%m-%d)
HOSTNAME=$(hostname)
IP_ADDRESS=$(curl -s ifconfig.me)
FILE_COUNT=$(find /var/log -name "*.log" | wc -l)

# Environment variables
export MY_VAR="value"               # Available to child processes
echo "$HOME"                        # /home/username
echo "$USER"                        # Current username
echo "$PWD"                         # Current directory
echo "$PATH"                        # Executable search path
echo "$SHELL"                       # Current shell

# Default values
DB_HOST="${DB_HOST:-localhost}"      # Use localhost if DB_HOST is not set
TIMEOUT="${TIMEOUT:-30}"            # Default 30 if not set

# Read-only (constants)
readonly MAX_RETRIES=5

# Arrays
SERVERS=("web1" "web2" "web3" "db1")
echo "${SERVERS[0]}"                # First element: web1
echo "${SERVERS[@]}"                # All elements
echo "${#SERVERS[@]}"               # Array length: 4
SERVERS+=("web4")                   # Append to array

# Special variables
$0    # Script name
$1    # First argument
$2    # Second argument
$#    # Number of arguments
$@    # All arguments (individually quoted)
$*    # All arguments (as single string)
$?    # Exit code of last command (0 = success)
$$    # Current script's PID
$!    # PID of last background process
```

---

## 4.4 Conditionals

```bash
# Basic if-elif-else
if [ "$ENVIRONMENT" == "production" ]; then
    echo "Running in production mode"
elif [ "$ENVIRONMENT" == "staging" ]; then
    echo "Running in staging mode"
else
    echo "Running in development mode"
fi

# File tests
[ -f /path/file ]       # File exists
[ -d /path/dir ]        # Directory exists
[ -r file ]             # File is readable
[ -w file ]             # File is writable
[ -x file ]             # File is executable
[ -s file ]             # File exists and is not empty
[ ! -f file ]           # File does NOT exist

# String tests
[ -z "$var" ]           # String is empty (zero length)
[ -n "$var" ]           # String is NOT empty
[ "$a" == "$b" ]        # Strings are equal
[ "$a" != "$b" ]        # Strings are not equal

# Numeric comparison
[ "$a" -eq "$b" ]       # Equal
[ "$a" -ne "$b" ]       # Not equal
[ "$a" -gt "$b" ]       # Greater than
[ "$a" -lt "$b" ]       # Less than
[ "$a" -ge "$b" ]       # Greater than or equal
[ "$a" -le "$b" ]       # Less than or equal

# Combining conditions
[ "$a" -gt 5 ] && [ "$a" -lt 10 ]    # AND
[ "$a" == "yes" ] || [ "$a" == "y" ]  # OR

# Modern test syntax (preferred)
[[ "$name" == dev* ]]                  # Pattern matching
[[ "$age" -gt 18 && "$age" -lt 65 ]]  # Combined in one bracket

# Practical examples
if [[ ! -f "$CONFIG_FILE" ]]; then
    echo "ERROR: Config file not found: $CONFIG_FILE"
    exit 1
fi

if ! command -v docker &> /dev/null; then
    echo "Docker is not installed. Installing..."
    curl -fsSL https://get.docker.com | sh
fi
```

---

## 4.5 Loops

```bash
# For loop — iterate over list
for server in web1 web2 web3; do
    echo "Deploying to $server..."
    ssh "$server" "sudo systemctl restart app"
done

# For loop — iterate over array
SERVICES=("nginx" "redis" "postgres" "app")
for svc in "${SERVICES[@]}"; do
    if systemctl is-active --quiet "$svc"; then
        echo "✓ $svc is running"
    else
        echo "✗ $svc is DOWN!"
    fi
done

# For loop — range
for i in {1..10}; do
    echo "Iteration $i"
done

# For loop — C-style
for ((i=0; i<5; i++)); do
    echo "Counter: $i"
done

# While loop
count=0
while [ "$count" -lt 5 ]; do
    echo "Attempt $count"
    count=$((count + 1))
done

# While loop — read file line by line
while IFS= read -r line; do
    echo "Processing: $line"
done < servers.txt

# While loop — infinite with break
while true; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://myapp.com/health)
    if [ "$STATUS" == "200" ]; then
        echo "App is healthy!"
        break
    fi
    echo "Waiting for app to be ready..."
    sleep 5
done

# Until loop (opposite of while — runs until condition is TRUE)
until kubectl get pods | grep -q "Running"; do
    echo "Waiting for pods to start..."
    sleep 10
done
```

---

## 4.6 Functions

```bash
# Define function
greet() {
    local name=$1                    # local = scoped to function
    local greeting="${2:-Hello}"     # Default value
    echo "${greeting}, ${name}!"
}

# Call function
greet "World"                        # Hello, World!
greet "DevOps" "Welcome"             # Welcome, DevOps!

# Function with return value
is_service_running() {
    local service=$1
    if systemctl is-active --quiet "$service"; then
        return 0    # Success (true)
    else
        return 1    # Failure (false)
    fi
}

# Use return value
if is_service_running "nginx"; then
    echo "Nginx is running"
fi

# Function returning data (via echo + command substitution)
get_disk_usage() {
    local path="${1:-/}"
    df -h "$path" | awk 'NR==2 {print $5}' | tr -d '%'
}

USAGE=$(get_disk_usage "/")
if [ "$USAGE" -gt 80 ]; then
    echo "WARNING: Disk usage is ${USAGE}%"
fi

# Practical function library
log() {
    local level="${1:-INFO}"
    local message="$2"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [${level}] ${message}"
}

die() {
    log "ERROR" "$1"
    exit "${2:-1}"
}

# Usage
log "INFO" "Starting deployment"
log "WARN" "Disk space is low"
die "Cannot connect to database" 2
```

---

## 4.7 Error Handling & Debugging

```bash
# Trap — run cleanup on exit/error
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/myapp_*
}
trap cleanup EXIT          # Run cleanup on script exit (success or failure)
trap 'echo "Error on line $LINENO"; exit 1' ERR  # Run on any error

# Retry function
retry() {
    local max_attempts="${1:-3}"
    local delay="${2:-5}"
    local command="${@:3}"
    local attempt=1

    until $command; do
        if [ "$attempt" -ge "$max_attempts" ]; then
            echo "FAILED: '$command' after $max_attempts attempts"
            return 1
        fi
        echo "Attempt $attempt failed. Retrying in ${delay}s..."
        sleep "$delay"
        attempt=$((attempt + 1))
    done
}

# Usage
retry 5 10 curl -f http://myapp.com/health

# Input validation
validate_args() {
    if [ $# -lt 2 ]; then
        echo "Usage: $0 <environment> <version>"
        echo "Example: $0 production 2.1.0"
        exit 1
    fi

    local env=$1
    if [[ ! "$env" =~ ^(dev|staging|production)$ ]]; then
        echo "ERROR: Invalid environment '$env'"
        echo "Must be: dev, staging, or production"
        exit 1
    fi
}

validate_args "$@"

# Debugging
bash -x script.sh          # Print every command before executing
set -x                     # Enable debug mode inside script
set +x                     # Disable debug mode
```

---

## 4.8 Complete Practical Scripts

### Server Health Check Script
```bash
#!/bin/bash
set -euo pipefail

# Configuration
SERVICES=("nginx" "docker" "redis-server")
DISK_THRESHOLD=80
MEMORY_THRESHOLD=90
SLACK_WEBHOOK="${SLACK_WEBHOOK:-}"

# Functions
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"; }

check_services() {
    local failed=()
    for svc in "${SERVICES[@]}"; do
        if ! systemctl is-active --quiet "$svc" 2>/dev/null; then
            failed+=("$svc")
        fi
    done
    if [ ${#failed[@]} -gt 0 ]; then
        log "CRITICAL: Services down: ${failed[*]}"
        return 1
    fi
    log "OK: All services running"
    return 0
}

check_disk() {
    local usage
    usage=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
    if [ "$usage" -gt "$DISK_THRESHOLD" ]; then
        log "WARNING: Disk usage at ${usage}%"
        return 1
    fi
    log "OK: Disk usage at ${usage}%"
    return 0
}

check_memory() {
    local usage
    usage=$(free | awk '/Mem:/ {printf "%d", $3/$2 * 100}')
    if [ "$usage" -gt "$MEMORY_THRESHOLD" ]; then
        log "WARNING: Memory usage at ${usage}%"
        return 1
    fi
    log "OK: Memory usage at ${usage}%"
    return 0
}

# Main
log "=== Health Check Started ==="
ISSUES=0
check_services || ISSUES=$((ISSUES + 1))
check_disk || ISSUES=$((ISSUES + 1))
check_memory || ISSUES=$((ISSUES + 1))

if [ "$ISSUES" -gt 0 ]; then
    log "RESULT: $ISSUES issue(s) detected"
    exit 1
fi
log "RESULT: All checks passed"
```

### Automated Deployment Script
```bash
#!/bin/bash
set -euo pipefail

# Arguments
ENVIRONMENT="${1:-}"
VERSION="${2:-}"
APP_NAME="myapp"
DEPLOY_DIR="/opt/${APP_NAME}"

# Validation
if [ -z "$ENVIRONMENT" ] || [ -z "$VERSION" ]; then
    echo "Usage: $0 <environment> <version>"
    echo "Example: $0 production 2.1.0"
    exit 1
fi

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$ENVIRONMENT] $1"; }

# Confirm production deployments
if [ "$ENVIRONMENT" == "production" ]; then
    read -p "Deploy v${VERSION} to PRODUCTION? (yes/no): " confirm
    if [ "$confirm" != "yes" ]; then
        log "Deployment cancelled"
        exit 0
    fi
fi

log "Starting deployment of v${VERSION}"

# Pull new image
log "Pulling Docker image..."
docker pull "registry.company.com/${APP_NAME}:${VERSION}"

# Stop old version
log "Stopping current version..."
docker stop "${APP_NAME}" 2>/dev/null || true
docker rm "${APP_NAME}" 2>/dev/null || true

# Start new version
log "Starting v${VERSION}..."
docker run -d \
    --name "${APP_NAME}" \
    --restart unless-stopped \
    -p 8080:8080 \
    -e ENVIRONMENT="${ENVIRONMENT}" \
    --health-cmd="curl -f http://localhost:8080/health || exit 1" \
    --health-interval=10s \
    "registry.company.com/${APP_NAME}:${VERSION}"

# Wait for healthy
log "Waiting for health check..."
for i in {1..30}; do
    STATUS=$(docker inspect --format='{{.State.Health.Status}}' "${APP_NAME}" 2>/dev/null || echo "starting")
    if [ "$STATUS" == "healthy" ]; then
        log "Deployment successful! v${VERSION} is healthy."
        exit 0
    fi
    sleep 2
done

log "ERROR: Application failed to become healthy"
log "Rolling back..."
docker stop "${APP_NAME}"
docker rm "${APP_NAME}"
# Restart previous version logic here
exit 1
```

---

## 4.9 Shell Scripting Free Resources

| Resource | Link | Notes |
|----------|------|-------|
| ShellCheck | https://www.shellcheck.net | Paste scripts to find bugs (MUST USE) |
| Bash Academy | https://guide.bash.academy | Interactive Bash tutorial |
| Advanced Bash Scripting Guide | https://tldp.org/LDP/abs/html/ | Comprehensive reference |
| ExplainShell | https://explainshell.com | Explain any command |
| Bash Hackers Wiki | https://wiki.bash-hackers.org | Advanced techniques |
| Google Shell Style Guide | https://google.github.io/styleguide/shellguide.html | Best practices |
| DevHints Bash | https://devhints.io/bash | Quick cheat sheet |

---

## Month 1 Summary & Practice Exercises

### Key Takeaways
1. Linux is your daily workspace — master the terminal
2. Everything is a file in Linux (even processes and devices)
3. Networking knowledge separates good DevOps from great DevOps
4. Git is the foundation of ALL modern DevOps practices
5. Shell scripts are your first automation tool — keep them simple and safe

### Practice Exercises
```
Exercise 1: Set up an Ubuntu server (VM or cloud), configure:
  - Non-root user with SSH key authentication
  - Firewall (ufw) allowing only SSH and HTTP
  - Nginx web server serving a static page
  - Cron job that checks disk space every hour

Exercise 2: Write a bash script that:
  - Accepts a directory path as argument
  - Finds all log files older than 7 days
  - Compresses them with gzip
  - Moves them to an archive directory
  - Logs actions to a file

Exercise 3: Create a Git repository with:
  - .gitignore file
  - Branch protection on main
  - Feature branch workflow
  - Practice resolving a merge conflict
  - Tag a release

Exercise 4: Network troubleshooting:
  - Set up two Docker containers
  - Make them communicate via custom network
  - Troubleshoot when one can't reach the other
  - Use tcpdump to capture traffic between them
```
