# Month 3 - Part 2: Ansible Configuration Management

## DevOps Upskilling Program | GCC Job Market Focus

---

## 1. What is Configuration Management? Why Ansible?

Configuration Management (CM) is the practice of automating and managing the state of your infrastructure. Instead of manually configuring servers, you define the desired state in code and let a tool enforce it.

### Ansible vs Puppet vs Chef vs SaltStack

| Feature | Ansible | Puppet | Chef | SaltStack |
|---------|---------|--------|------|-----------|
| Language | YAML (Playbooks) | Puppet DSL (Ruby) | Ruby DSL | YAML/Jinja2 |
| Architecture | Agentless (SSH) | Agent-based | Agent-based | Agent-based (or agentless) |
| Model | Push | Pull | Pull | Push/Pull |
| Learning Curve | Low | Medium | High | Medium |
| Popularity in GCC | Very High | Medium | Low | Low |
| Community | Massive | Large | Medium | Medium |
| Idempotent | Yes | Yes | Yes | Yes |
| Commercial | AAP (Red Hat) | Puppet Enterprise | Chef Automate | SaltStack Enterprise |

### Agentless Architecture (SSH)

Ansible uses SSH (Linux) or WinRM (Windows) to connect to managed nodes. No agent software needs to be installed on target machines.

```
┌──────────────┐         SSH          ┌──────────────┐
│ Control Node │ ──────────────────── │ Managed Node │
│ (Ansible)    │                      │ (No Agent)   │
└──────────────┘         SSH          ┌──────────────┐
                  ──────────────────── │ Managed Node │
                                      │ (No Agent)   │
                                      └──────────────┘
```

**Benefits:**
- No agent to install, update, or secure
- No listening ports on managed nodes
- No PKI infrastructure required
- Works with any machine you can SSH into

### Push vs Pull Model

**Push Model (Ansible):**
- Control node pushes configuration to managed nodes on demand
- You decide when changes are applied
- Immediate feedback on success/failure

**Pull Model (Puppet/Chef):**
- Agents on each node periodically pull configuration from a central server
- Changes apply on next agent run (every 30 minutes by default)
- Requires always-on central server

### Idempotency Explained

Idempotency means running the same operation multiple times produces the same result. Ansible modules are designed to be idempotent — they only make changes when the current state differs from the desired state.

```yaml
# This task is idempotent - running it 100 times still results in one file
- name: Ensure config file exists
  ansible.builtin.copy:
    src: app.conf
    dest: /etc/app/app.conf
    owner: root
    mode: '0644'
```

**Non-idempotent example (avoid):**
```yaml
# BAD - This appends a line every time it runs
- name: Add DNS server
  ansible.builtin.shell: echo "nameserver 8.8.8.8" >> /etc/resolv.conf

# GOOD - Idempotent alternative
- name: Ensure DNS server is configured
  ansible.builtin.lineinfile:
    path: /etc/resolv.conf
    line: "nameserver 8.8.8.8"
    state: present
```

---

## 2. Ansible Architecture

### Control Node vs Managed Nodes

**Control Node:** The machine where Ansible is installed and playbooks are executed from.
- Must be Linux/macOS (Windows not supported as control node)
- Requires Python 3.9+
- Has ansible package installed

**Managed Nodes:** Target machines that Ansible configures.
- Require Python (for most modules)
- Require SSH access (Linux) or WinRM (Windows)
- No Ansible installation needed

### Inventory (Static, Dynamic)

Inventory defines the hosts Ansible manages. Can be a simple file or a dynamic script/plugin that queries cloud APIs.

### Modules, Plugins, Collections

- **Modules:** Units of work (e.g., `apt`, `copy`, `service`). ~6000+ available.
- **Plugins:** Extend Ansible's core functionality (connection, callback, lookup, filter plugins).
- **Collections:** Packaging format that bundles modules, plugins, roles, and playbooks. Distributed via Ansible Galaxy.

### Ansible Configuration (ansible.cfg)

```ini
# ansible.cfg - searched in this order:
# 1. ANSIBLE_CONFIG environment variable
# 2. ./ansible.cfg (current directory)
# 3. ~/.ansible.cfg (home directory)
# 4. /etc/ansible/ansible.cfg (global)

[defaults]
inventory = ./inventory/hosts.yml
remote_user = ansible
private_key_file = ~/.ssh/ansible_key
host_key_checking = False
retry_files_enabled = False
stdout_callback = yaml
timeout = 30
forks = 20

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

### Connection Methods

| Method | Use Case |
|--------|----------|
| ssh | Default for Linux hosts |
| winrm | Windows hosts |
| local | Run on control node itself |
| docker | Connect to Docker containers |
| network_cli | Network devices (Cisco, Juniper) |

---

## 3. Inventory Deep Dive

### INI Format

```ini
# inventory/hosts.ini

# Ungrouped hosts
jump-server ansible_host=10.0.0.1

[webservers]
web1 ansible_host=10.0.1.10
web2 ansible_host=10.0.1.11
web3 ansible_host=10.0.1.12

[dbservers]
db1 ansible_host=10.0.2.10
db2 ansible_host=10.0.2.11

[loadbalancers]
lb1 ansible_host=10.0.0.50

# Group of groups
[production:children]
webservers
dbservers
loadbalancers

# Group variables
[webservers:vars]
http_port=80
app_env=production

[all:vars]
ansible_user=deploy
ansible_python_interpreter=/usr/bin/python3
```

### YAML Format

```yaml
# inventory/hosts.yml
all:
  vars:
    ansible_user: deploy
    ansible_python_interpreter: /usr/bin/python3
  hosts:
    jump-server:
      ansible_host: 10.0.0.1
  children:
    webservers:
      vars:
        http_port: 80
        app_env: production
      hosts:
        web1:
          ansible_host: 10.0.1.10
        web2:
          ansible_host: 10.0.1.11
        web3:
          ansible_host: 10.0.1.12
    dbservers:
      hosts:
        db1:
          ansible_host: 10.0.2.10
          db_role: primary
        db2:
          ansible_host: 10.0.2.11
          db_role: replica
    loadbalancers:
      hosts:
        lb1:
          ansible_host: 10.0.0.50
    production:
      children:
        webservers:
        dbservers:
        loadbalancers:
```

### Host and Group Variables (File-based)

```
inventory/
├── hosts.yml
├── group_vars/
│   ├── all.yml          # applies to all hosts
│   ├── webservers.yml   # applies to webservers group
│   └── dbservers.yml    # applies to dbservers group
└── host_vars/
    ├── web1.yml         # applies only to web1
    └── db1.yml          # applies only to db1
```

```yaml
# inventory/group_vars/webservers.yml
---
nginx_version: "1.24"
nginx_worker_processes: auto
nginx_worker_connections: 1024
ssl_certificate_path: /etc/ssl/certs/app.crt
```

### Dynamic Inventory (AWS EC2)

```yaml
# inventory/aws_ec2.yml
---
plugin: amazon.aws.aws_ec2
regions:
  - me-south-1    # Bahrain (GCC region)
  - me-central-1  # UAE

filters:
  tag:Environment:
    - production
    - staging
  instance-state-name: running

keyed_groups:
  - key: tags.Role
    prefix: role
  - key: placement.region
    prefix: region
  - key: tags.Environment
    prefix: env

hostnames:
  - private-ip-address

compose:
  ansible_host: private_ip_address
  ansible_user: "'ubuntu'"
```

### Dynamic Inventory (Azure)

```yaml
# inventory/azure_rm.yml
---
plugin: azure.azcollection.azure_rm
auth_source: auto
include_vm_resource_groups:
  - production-rg
  - staging-rg

keyed_groups:
  - prefix: tag
    key: tags
  - prefix: location
    key: location

hostnames:
  - private_ip_address
```

### Patterns for Targeting Hosts

```bash
# All hosts
ansible all -m ping

# Specific group
ansible webservers -m ping

# Multiple groups (OR - union)
ansible 'webservers:dbservers' -m ping

# Intersection (AND)
ansible 'webservers:&production' -m ping

# Exclusion (NOT)
ansible 'all:!loadbalancers' -m ping

# Wildcard
ansible 'web*' -m ping

# Regex
ansible '~web[0-9]+' -m ping

# Specific host
ansible web1 -m ping

# First host in group
ansible 'webservers[0]' -m ping
```

---

## 4. Ad-Hoc Commands

Ad-hoc commands are one-liner Ansible commands for quick tasks without writing a playbook.

### Syntax

```bash
ansible <pattern> -m <module> -a "<arguments>" [options]
```

### Common Modules with Ad-Hoc

```bash
# Ping all hosts (connectivity test)
ansible all -m ping

# Run a shell command
ansible webservers -m shell -a "df -h /var"

# Run a command (no shell features like pipes/redirects)
ansible webservers -m command -a "uptime"

# Copy a file
ansible webservers -m copy -a "src=./app.conf dest=/etc/app/ mode=0644" -b

# Manage files/directories
ansible webservers -m file -a "path=/opt/app state=directory mode=0755 owner=app" -b

# Install a package (Debian/Ubuntu)
ansible webservers -m apt -a "name=nginx state=present update_cache=yes" -b

# Install a package (RHEL/CentOS)
ansible webservers -m yum -a "name=httpd state=present" -b

# Manage a service
ansible webservers -m service -a "name=nginx state=started enabled=yes" -b

# Create a user
ansible all -m user -a "name=deploy state=present shell=/bin/bash groups=sudo" -b

# Gather facts
ansible web1 -m setup -a "filter=ansible_os_family"
```

### Useful Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-m` | Specify module | `-m apt` |
| `-a` | Module arguments | `-a "name=nginx state=present"` |
| `-b` | Become (sudo) | `ansible all -b -m apt -a "name=vim state=present"` |
| `-i` | Inventory file | `-i inventory/production.yml` |
| `--limit` | Limit to specific hosts | `--limit web1` |
| `-u` | Remote user | `-u deploy` |
| `-k` | Ask SSH password | |
| `-K` | Ask become password | |
| `--check` | Dry run | |
| `-f` | Forks (parallelism) | `-f 50` |
| `-v/-vv/-vvv` | Verbosity | |

### When to Use Ad-Hoc vs Playbooks

**Use Ad-Hoc for:**
- Quick checks (disk space, uptime, connectivity)
- One-time tasks (restart a service, kill a process)
- Gathering information
- Emergency fixes

**Use Playbooks for:**
- Repeatable processes
- Complex multi-step tasks
- Configuration that should be version controlled
- Anything you'll run more than once

---

## 5. Playbooks Deep Dive

### Playbook Structure

```yaml
---
# A playbook contains one or more plays
# Each play targets a group of hosts and defines tasks

- name: Configure web servers        # Play 1
  hosts: webservers
  become: yes
  gather_facts: yes
  vars:
    http_port: 80

  pre_tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes
        cache_valid_time: 3600

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
      notify: Restart nginx

    - name: Deploy configuration
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted

  post_tasks:
    - name: Verify nginx is running
      ansible.builtin.uri:
        url: "http://localhost:{{ http_port }}"
        status_code: 200

- name: Configure database servers    # Play 2
  hosts: dbservers
  become: yes
  tasks:
    - name: Install PostgreSQL
      ansible.builtin.apt:
        name: postgresql
        state: present
```

### Task Attributes

```yaml
tasks:
  - name: Install packages            # Human-readable description
    ansible.builtin.apt:
      name: "{{ item }}"
      state: present
    loop:                              # Iterate over a list
      - nginx
      - certbot
      - python3-certbot-nginx
    register: install_result           # Store task output
    when: ansible_os_family == "Debian"  # Conditional execution
    notify: Restart nginx              # Trigger handler on change
    tags:                              # Tags for selective execution
      - packages
      - nginx
    ignore_errors: no                  # Default: fail on error
    changed_when: install_result.changed  # Custom change detection
    failed_when: "'FAILED' in install_result.stderr"  # Custom failure
    become: yes                        # Run with sudo
    timeout: 300                       # Task timeout in seconds
```

### Common Modules with Full Examples

```yaml
# --- File Management ---
- name: Create a directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: app
    group: app
    mode: '0755'

- name: Create a symlink
  ansible.builtin.file:
    src: /opt/myapp/current/config.yml
    dest: /etc/myapp/config.yml
    state: link

- name: Copy file with validation
  ansible.builtin.copy:
    src: sudoers_deploy
    dest: /etc/sudoers.d/deploy
    owner: root
    mode: '0440'
    validate: /usr/sbin/visudo -cf %s

- name: Deploy template
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
    backup: yes
  notify: Reload nginx

- name: Ensure line exists in file
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'
    validate: '/usr/sbin/sshd -t -f %s'
  notify: Restart sshd

# --- Package Management ---
- name: Install multiple packages (apt)
  ansible.builtin.apt:
    name:
      - nginx
      - certbot
      - fail2ban
    state: present
    update_cache: yes
    cache_valid_time: 3600

- name: Install package (yum)
  ansible.builtin.yum:
    name: httpd
    state: latest
    enablerepo: epel

- name: Install Python packages
  ansible.builtin.pip:
    name:
      - flask==2.3.0
      - gunicorn==21.2.0
    virtualenv: /opt/myapp/venv
    virtualenv_command: python3 -m venv

# --- Service Management ---
- name: Start and enable service
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes

- name: Restart via systemd with daemon reload
  ansible.builtin.systemd:
    name: myapp
    state: restarted
    daemon_reload: yes
    enabled: yes

# --- Command Execution ---
- name: Run command (simple, no shell)
  ansible.builtin.command:
    cmd: /opt/myapp/bin/migrate
    chdir: /opt/myapp
    creates: /opt/myapp/.migrated    # Skip if file exists

- name: Run shell command (pipes, redirects allowed)
  ansible.builtin.shell:
    cmd: |
      cat /var/log/app.log | grep ERROR | wc -l
  register: error_count
  changed_when: false

- name: Run a script from control node on managed nodes
  ansible.builtin.script:
    cmd: scripts/setup.sh --env production
  args:
    creates: /opt/app/.initialized

- name: Raw command (no Python required on target)
  ansible.builtin.raw: apt-get install -y python3
  when: ansible_python_interpreter is not defined

# --- User Management ---
- name: Create application user
  ansible.builtin.user:
    name: appuser
    uid: 1050
    group: app
    shell: /bin/bash
    home: /opt/appuser
    create_home: yes
    generate_ssh_key: yes

- name: Create group
  ansible.builtin.group:
    name: app
    gid: 1050
    state: present

- name: Add SSH authorized key
  ansible.posix.authorized_key:
    user: deploy
    state: present
    key: "{{ lookup('file', '~/.ssh/deploy_key.pub') }}"

# --- Downloads and Git ---
- name: Clone repository
  ansible.builtin.git:
    repo: https://github.com/org/myapp.git
    dest: /opt/myapp
    version: v2.1.0
    force: yes

- name: Download and extract archive
  ansible.builtin.unarchive:
    src: https://releases.example.com/app-v2.1.tar.gz
    dest: /opt/
    remote_src: yes

- name: Download file
  ansible.builtin.get_url:
    url: https://releases.example.com/app-v2.1.tar.gz
    dest: /tmp/app-v2.1.tar.gz
    checksum: sha256:abc123...
    mode: '0644'

# --- Docker ---
- name: Pull Docker image
  community.docker.docker_image:
    name: nginx
    tag: "1.25-alpine"
    source: pull

- name: Run Docker container
  community.docker.docker_container:
    name: myapp
    image: "myregistry.com/myapp:{{ app_version }}"
    state: started
    restart_policy: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /opt/myapp/data:/app/data
    env:
      DATABASE_URL: "{{ db_connection_string }}"
      APP_ENV: production

# --- API Calls ---
- name: Call an API endpoint
  ansible.builtin.uri:
    url: "https://api.example.com/health"
    method: GET
    headers:
      Authorization: "Bearer {{ api_token }}"
    status_code: 200
    return_content: yes
  register: api_response

- name: Create resource via API
  ansible.builtin.uri:
    url: "https://api.example.com/resources"
    method: POST
    body_format: json
    body:
      name: "my-resource"
      type: "compute"
    status_code: [200, 201]

# --- Debugging and Flow Control ---
- name: Print variable value
  ansible.builtin.debug:
    msg: "The app version is {{ app_version }}"

- name: Print variable structure
  ansible.builtin.debug:
    var: api_response.json

- name: Assert conditions are met
  ansible.builtin.assert:
    that:
      - ansible_memtotal_mb >= 2048
      - ansible_processor_vcpus >= 2
    fail_msg: "Server does not meet minimum requirements"
    success_msg: "Server meets all requirements"

- name: Fail with message
  ansible.builtin.fail:
    msg: "Deployment aborted: health check failed"
  when: health_check.status != 200

- name: Pause for confirmation
  ansible.builtin.pause:
    prompt: "Press Enter to continue with deployment to production"
```

### Blocks (Error Handling)

```yaml
tasks:
  - name: Deploy application with error handling
    block:
      - name: Pull latest code
        ansible.builtin.git:
          repo: https://github.com/org/app.git
          dest: /opt/app
          version: "{{ release_version }}"

      - name: Run database migrations
        ansible.builtin.command:
          cmd: python manage.py migrate
          chdir: /opt/app

      - name: Restart application
        ansible.builtin.systemd:
          name: myapp
          state: restarted

    rescue:
      - name: Rollback to previous version
        ansible.builtin.git:
          repo: https://github.com/org/app.git
          dest: /opt/app
          version: "{{ previous_version }}"

      - name: Restart with old version
        ansible.builtin.systemd:
          name: myapp
          state: restarted

      - name: Notify team of failure
        ansible.builtin.uri:
          url: "{{ slack_webhook }}"
          method: POST
          body_format: json
          body:
            text: "Deployment of {{ release_version }} FAILED. Rolled back."

    always:
      - name: Clean up temp files
        ansible.builtin.file:
          path: /tmp/deploy-artifacts
          state: absent

      - name: Log deployment attempt
        ansible.builtin.lineinfile:
          path: /var/log/deployments.log
          line: "{{ ansible_date_time.iso8601 }} - {{ release_version }} - {{ 'FAILED' if ansible_failed_task is defined else 'SUCCESS' }}"
          create: yes
```

### Import vs Include

```yaml
# import_tasks - static, processed at playbook parse time
# include_tasks - dynamic, processed at runtime

# Static import (variables/conditionals resolved at parse time)
- name: Import common tasks
  ansible.builtin.import_tasks: tasks/common.yml

# Dynamic include (can use variables, loops, conditionals at runtime)
- name: Include OS-specific tasks
  ansible.builtin.include_tasks: "tasks/{{ ansible_os_family | lower }}.yml"

# Import a playbook (only at play level)
- name: Import another playbook
  ansible.builtin.import_playbook: playbooks/base-setup.yml

# Include role dynamically
- name: Include role based on condition
  ansible.builtin.include_role:
    name: "{{ database_type }}"
  when: database_type is defined
```

**Key Differences:**

| Feature | import_* | include_* |
|---------|----------|-----------|
| Processing | Parse time (static) | Runtime (dynamic) |
| Loops | Cannot use loops | Can use loops |
| Variables in filename | No | Yes |
| Tags | Inherited by imported tasks | NOT inherited |
| Handlers | Can notify imported handlers | Cannot notify |

---

## 6. Variables & Facts

### Variable Precedence (Simplified - Highest to Lowest)

```
1.  extra vars (-e "var=value")              ← HIGHEST
2.  include params
3.  role (and include_role) params
4.  set_facts / registered vars
5.  block vars (inside block/rescue/always)
6.  task vars (only for the task)
7.  play vars_files
8.  play vars_prompt
9.  play vars
10. host_vars/host.yml
11. inventory host_vars
12. host_vars/group_vars (child group)
13. group_vars/group.yml
14. inventory group_vars (child group)
15. group_vars/all.yml
16. inventory group_vars/all
17. role defaults (defaults/main.yml)        ← LOWEST
```

**Rule of thumb:** Extra vars always win. Role defaults are the easiest to override.

### Defining Variables

```yaml
# In playbook
- hosts: webservers
  vars:
    app_name: myapp
    app_port: 8080
    app_env: production

# From vars_files
- hosts: webservers
  vars_files:
    - vars/common.yml
    - "vars/{{ ansible_os_family }}.yml"

# Registered variables (capture task output)
- name: Check disk space
  ansible.builtin.command: df -h /var
  register: disk_info
  changed_when: false

- name: Show disk info
  ansible.builtin.debug:
    var: disk_info.stdout_lines

# Extra vars (command line - highest precedence)
# ansible-playbook site.yml -e "app_version=2.1.0 deploy_env=production"

# Set_fact (runtime variable creation)
- name: Calculate deployment path
  ansible.builtin.set_fact:
    deploy_path: "/opt/{{ app_name }}/releases/{{ app_version }}"
    deploy_timestamp: "{{ ansible_date_time.iso8601 }}"
```

### Ansible Facts

```yaml
# Facts are auto-gathered system information
- hosts: all
  gather_facts: yes    # Default: yes
  tasks:
    - name: Show OS info
      ansible.builtin.debug:
        msg: |
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          IP: {{ ansible_default_ipv4.address }}
          RAM: {{ ansible_memtotal_mb }} MB
          CPUs: {{ ansible_processor_vcpus }}
          Hostname: {{ ansible_fqdn }}

# Custom facts (place on managed nodes)
# /etc/ansible/facts.d/app.fact (INI or JSON)
```

```ini
; /etc/ansible/facts.d/app.fact
[general]
app_name=myapp
version=2.1.0
environment=production
```

```yaml
# Access custom facts
- name: Show custom fact
  ansible.builtin.debug:
    msg: "App version: {{ ansible_local.app.general.version }}"
```

### Magic Variables

```yaml
- name: Demonstrate magic variables
  ansible.builtin.debug:
    msg: |
      Current host: {{ inventory_hostname }}
      Short name: {{ inventory_hostname_short }}
      All groups this host belongs to: {{ group_names }}
      All hosts in webservers: {{ groups['webservers'] }}
      Variable from another host: {{ hostvars['db1']['db_port'] }}
      All hosts in current play: {{ play_hosts }}
      Current role path: {{ role_path }}
      Playbook directory: {{ playbook_dir }}
```

### Vault-Encrypted Variables

```yaml
# vars/secrets.yml (encrypted with ansible-vault)
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  6539306538613338...
api_key: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  3261643539316234...
```

---

## 7. Jinja2 Templates

### Template Syntax

```jinja2
{# This is a comment #}

{# Variables #}
server_name {{ server_name }};

{# Conditionals #}
{% if enable_ssl %}
listen 443 ssl;
ssl_certificate {{ ssl_cert_path }};
{% else %}
listen 80;
{% endif %}

{# Loops #}
{% for server in backend_servers %}
server {{ server.host }}:{{ server.port }} weight={{ server.weight | default(1) }};
{% endfor %}

{# Filters #}
{{ my_list | join(', ') }}
{{ my_string | upper }}
{{ my_var | default('fallback_value') }}
```

### Common Filters

```yaml
# default - provide fallback value
"{{ my_var | default('none') }}"

# join - join list into string
"{{ my_list | join(', ') }}"

# map - apply filter to each item
"{{ users | map(attribute='name') | list }}"

# select/reject - filter items
"{{ ports | select('gt', 1024) | list }}"

# regex_replace
"{{ hostname | regex_replace('^web', 'app') }}"

# to_json / to_yaml
"{{ my_dict | to_nice_json }}"

# ipaddr - IP address manipulation
"{{ my_ip | ansible.utils.ipaddr('network') }}"

# combine - merge dictionaries
"{{ defaults | combine(overrides) }}"

# ternary - inline if/else
"{{ 'yes' if enable_feature else 'no' }}"
```

### Real-World Template: nginx.conf.j2

```jinja2
# {{ ansible_managed }}
# Do not edit this file manually

user {{ nginx_user | default('www-data') }};
worker_processes {{ nginx_worker_processes | default('auto') }};
pid /run/nginx.pid;

events {
    worker_connections {{ nginx_worker_connections | default(1024) }};
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

{% for vhost in nginx_vhosts %}
    server {
        listen {{ vhost.port | default(80) }};
        server_name {{ vhost.server_name }};
        root {{ vhost.document_root }};

{% if vhost.ssl | default(false) %}
        listen 443 ssl;
        ssl_certificate {{ vhost.ssl_cert }};
        ssl_certificate_key {{ vhost.ssl_key }};
{% endif %}

{% if vhost.locations is defined %}
{% for location in vhost.locations %}
        location {{ location.path }} {
            proxy_pass {{ location.proxy_pass }};
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
{% endfor %}
{% endif %}
    }
{% endfor %}
}
```

### Real-World Template: docker-compose.yml.j2

```jinja2
# {{ ansible_managed }}
version: '3.8'

services:
{% for service in docker_services %}
  {{ service.name }}:
    image: {{ service.image }}:{{ service.tag | default('latest') }}
    restart: {{ service.restart_policy | default('unless-stopped') }}
{% if service.ports is defined %}
    ports:
{% for port in service.ports %}
      - "{{ port }}"
{% endfor %}
{% endif %}
{% if service.environment is defined %}
    environment:
{% for key, value in service.environment.items() %}
      {{ key }}: "{{ value }}"
{% endfor %}
{% endif %}
{% if service.volumes is defined %}
    volumes:
{% for volume in service.volumes %}
      - {{ volume }}
{% endfor %}
{% endif %}

{% endfor %}
{% if docker_networks is defined %}
networks:
{% for network in docker_networks %}
  {{ network.name }}:
    driver: {{ network.driver | default('bridge') }}
{% endfor %}
{% endif %}
```

---

## 8. Roles

### Why Roles?

Roles provide a structured way to organize playbooks into reusable components. They enforce a standard directory layout and make sharing automation easy.

### Role Directory Structure

```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml        # Main task list
    ├── handlers/
    │   └── main.yml        # Handler definitions
    ├── templates/
    │   └── nginx.conf.j2   # Jinja2 templates
    ├── files/
    │   └── index.html      # Static files
    ├── vars/
    │   └── main.yml        # High-priority variables
    ├── defaults/
    │   └── main.yml        # Default variables (lowest priority)
    ├── meta/
    │   └── main.yml        # Role metadata and dependencies
    └── README.md
```

### Creating Roles

```bash
# Create role skeleton
ansible-galaxy role init roles/webserver

# Install role from Galaxy
ansible-galaxy role install geerlingguy.docker

# Install roles from requirements file
ansible-galaxy role install -r requirements.yml
```

```yaml
# requirements.yml
---
roles:
  - name: geerlingguy.docker
    version: "6.1.0"
  - name: geerlingguy.certbot
    version: "5.0.0"

collections:
  - name: community.docker
    version: "3.4.0"
  - name: amazon.aws
    version: "6.5.0"
```

### Using Roles in Playbooks

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes

  roles:
    - common                          # Simple reference
    - role: webserver                  # With parameters
      vars:
        nginx_port: 8080
    - role: ssl                        # Conditional role
      when: enable_ssl | default(false)
```

### Role Dependencies (meta/main.yml)

```yaml
# roles/webserver/meta/main.yml
---
dependencies:
  - role: common
  - role: firewall
    vars:
      firewall_allowed_ports:
        - 80
        - 443
```

### Complete Role Example: Web Server

```yaml
# roles/webserver/defaults/main.yml
---
nginx_port: 80
nginx_worker_processes: auto
nginx_worker_connections: 1024
nginx_server_name: "_"
nginx_root: /var/www/html
nginx_user: www-data
app_health_endpoint: /health
```

```yaml
# roles/webserver/tasks/main.yml
---
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: yes
  tags: [nginx, packages]

- name: Deploy nginx configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
    validate: nginx -t -c %s
  notify: Reload nginx
  tags: [nginx, config]

- name: Deploy site configuration
  ansible.builtin.template:
    src: site.conf.j2
    dest: /etc/nginx/sites-available/default
    owner: root
    mode: '0644'
  notify: Reload nginx
  tags: [nginx, config]

- name: Ensure nginx is started and enabled
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
  tags: [nginx, service]

- name: Verify nginx is responding
  ansible.builtin.uri:
    url: "http://localhost:{{ nginx_port }}{{ app_health_endpoint }}"
    status_code: 200
  register: health_check
  retries: 3
  delay: 5
  tags: [nginx, verify]
```

```yaml
# roles/webserver/handlers/main.yml
---
- name: Reload nginx
  ansible.builtin.service:
    name: nginx
    state: reloaded

- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

---

## 9. Ansible Vault

### Encrypting Files

```bash
# Create a new encrypted file
ansible-vault create secrets.yml

# Encrypt an existing file
ansible-vault encrypt vars/production-secrets.yml

# Decrypt a file
ansible-vault decrypt vars/production-secrets.yml

# Edit an encrypted file
ansible-vault edit secrets.yml

# View encrypted file without decrypting
ansible-vault view secrets.yml

# Re-key (change password)
ansible-vault rekey secrets.yml
```

### Encrypting Strings

```bash
# Encrypt a single string
ansible-vault encrypt_string 'SuperS3cretP@ss!' --name 'db_password'

# Output (paste into your vars file):
# db_password: !vault |
#   $ANSIBLE_VAULT;1.1;AES256
#   623163656...
```

### Using Vault in Playbooks

```yaml
# vars/secrets.yml (encrypted)
---
db_password: "SuperS3cretP@ss!"
api_key: "sk-abc123def456"
ssl_private_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvgIBADANBg...
  -----END PRIVATE KEY-----
```

```yaml
# playbook referencing vault-encrypted file
---
- hosts: webservers
  vars_files:
    - vars/common.yml
    - vars/secrets.yml    # Encrypted file
  tasks:
    - name: Configure database connection
      ansible.builtin.template:
        src: db-config.j2
        dest: /etc/app/db.conf
        mode: '0600'
```

```bash
# Running with vault
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
ansible-playbook site.yml --vault-password-file /path/to/vault-script.sh
```

### Vault Password Files

```bash
# Create password file
echo 'MyVaultPassword123!' > ~/.vault_pass
chmod 600 ~/.vault_pass

# Reference in ansible.cfg
# [defaults]
# vault_password_file = ~/.vault_pass
```

### Multi-Vault IDs

```bash
# Encrypt with specific vault ID
ansible-vault encrypt --vault-id prod@prompt vars/prod-secrets.yml
ansible-vault encrypt --vault-id dev@~/.vault_pass_dev vars/dev-secrets.yml

# Run with multiple vault IDs
ansible-playbook site.yml \
  --vault-id dev@~/.vault_pass_dev \
  --vault-id prod@prompt
```

---

## 10. Error Handling & Debugging

### ignore_errors

```yaml
- name: Try to stop service (may not exist)
  ansible.builtin.service:
    name: old-app
    state: stopped
  ignore_errors: yes
  register: stop_result

- name: Handle missing service
  ansible.builtin.debug:
    msg: "Service was not running or doesn't exist"
  when: stop_result is failed
```

### failed_when / changed_when

```yaml
- name: Run database check
  ansible.builtin.command: /opt/app/bin/db-check
  register: db_check
  failed_when:
    - db_check.rc != 0
    - "'MAINTENANCE' not in db_check.stdout"
  changed_when: false   # Command never changes state

- name: Run migration
  ansible.builtin.command: python manage.py migrate --check
  register: migration
  changed_when: "'No migrations to apply' not in migration.stdout"
```

### block/rescue/always (see Section 5 for full example)

### Debugging Techniques

```yaml
# Debug module
- name: Print all variables for a host
  ansible.builtin.debug:
    var: vars

- name: Print specific variable
  ansible.builtin.debug:
    msg: "Value is: {{ my_var | type_debug }} = {{ my_var }}"
    verbosity: 2    # Only shows with -vv or higher

# Strategy: debug (interactive debugger)
# In playbook:
# strategy: debug
# Then at failure: p task.args, p result, redo, continue
```

```bash
# Verbosity levels
ansible-playbook site.yml -v      # Verbose
ansible-playbook site.yml -vv     # More verbose
ansible-playbook site.yml -vvv    # Connection debugging
ansible-playbook site.yml -vvvv   # Includes connection plugin details

# Check mode (dry run)
ansible-playbook site.yml --check --diff

# Step through tasks one at a time
ansible-playbook site.yml --step

# Start at a specific task
ansible-playbook site.yml --start-at-task="Install nginx"

# List tasks without executing
ansible-playbook site.yml --list-tasks

# List hosts that would be affected
ansible-playbook site.yml --list-hosts
```

---

## 11. Ansible + CI/CD Integration

### Running Ansible from GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - deploy

ansible-lint:
  stage: lint
  image: cytopia/ansible-lint:latest
  script:
    - ansible-lint playbooks/

deploy-staging:
  stage: deploy
  image: willhallonline/ansible:2.15-alpine
  variables:
    ANSIBLE_HOST_KEY_CHECKING: "False"
  before_script:
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' > ~/.ssh/id_rsa
    - chmod 600 ~/.ssh/id_rsa
    - echo "$VAULT_PASSWORD" > /tmp/.vault_pass
  script:
    - ansible-playbook -i inventory/staging.yml
        playbooks/deploy.yml
        --vault-password-file /tmp/.vault_pass
        -e "app_version=${CI_COMMIT_TAG}"
  after_script:
    - rm -f /tmp/.vault_pass ~/.ssh/id_rsa
  only:
    - tags
  environment:
    name: staging
```

### Running from Azure DevOps

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '3.11'

  - script: |
      pip install ansible==8.0.0 ansible-lint
    displayName: 'Install Ansible'

  - script: |
      mkdir -p ~/.ssh
      echo "$(SSH_PRIVATE_KEY)" > ~/.ssh/id_rsa
      chmod 600 ~/.ssh/id_rsa
      echo "$(VAULT_PASSWORD)" > .vault_pass
    displayName: 'Setup credentials'

  - script: |
      ansible-playbook -i inventory/production.yml \
        playbooks/deploy.yml \
        --vault-password-file .vault_pass \
        -e "app_version=$(Build.BuildNumber)"
    displayName: 'Run Ansible Playbook'
    env:
      ANSIBLE_HOST_KEY_CHECKING: false

  - script: |
      rm -f .vault_pass ~/.ssh/id_rsa
    displayName: 'Cleanup'
    condition: always()
```

### SSH Keys and Vault Password in CI

**Best practices:**
- Store SSH private keys as CI/CD secret variables
- Store vault password as a secret variable
- Never commit keys or passwords to the repository
- Use short-lived credentials where possible
- Clean up secrets in `after_script` or `always` conditions
- Use separate vault passwords for different environments

---

## 12. Complete Practical Playbooks

### Server Hardening Playbook

```yaml
---
- name: Server Hardening
  hosts: all
  become: yes
  vars:
    ssh_port: 22
    allowed_users:
      - deploy
      - admin
    sysctl_settings:
      net.ipv4.ip_forward: 0
      net.ipv4.conf.all.send_redirects: 0
      net.ipv4.conf.default.accept_source_route: 0

  tasks:
    - name: Update all packages
      ansible.builtin.apt:
        upgrade: dist
        update_cache: yes
        cache_valid_time: 3600

    - name: Install security packages
      ansible.builtin.apt:
        name:
          - ufw
          - fail2ban
          - unattended-upgrades
          - auditd
        state: present

    - name: Configure SSH hardening
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
        validate: '/usr/sbin/sshd -t -f %s'
      loop:
        - { regexp: '^#?PermitRootLogin', line: 'PermitRootLogin no' }
        - { regexp: '^#?PasswordAuthentication', line: 'PasswordAuthentication no' }
        - { regexp: '^#?X11Forwarding', line: 'X11Forwarding no' }
        - { regexp: '^#?MaxAuthTries', line: 'MaxAuthTries 3' }
        - { regexp: '^#?AllowUsers', line: "AllowUsers {{ allowed_users | join(' ') }}" }
      notify: Restart sshd

    - name: Configure UFW defaults
      community.general.ufw:
        direction: "{{ item.direction }}"
        policy: "{{ item.policy }}"
      loop:
        - { direction: incoming, policy: deny }
        - { direction: outgoing, policy: allow }

    - name: Allow SSH through firewall
      community.general.ufw:
        rule: allow
        port: "{{ ssh_port }}"
        proto: tcp

    - name: Enable UFW
      community.general.ufw:
        state: enabled

    - name: Apply sysctl settings
      ansible.posix.sysctl:
        name: "{{ item.key }}"
        value: "{{ item.value }}"
        sysctl_set: yes
        reload: yes
      loop: "{{ sysctl_settings | dict2items }}"

    - name: Set file permissions on sensitive files
      ansible.builtin.file:
        path: "{{ item }}"
        mode: '0600'
      loop:
        - /etc/shadow
        - /etc/gshadow

  handlers:
    - name: Restart sshd
      ansible.builtin.service:
        name: sshd
        state: restarted
```

### Docker Installation and Configuration

```yaml
---
- name: Install and Configure Docker
  hosts: docker_hosts
  become: yes
  vars:
    docker_users:
      - deploy
    docker_compose_version: "2.21.0"

  tasks:
    - name: Install prerequisites
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present
        update_cache: yes

    - name: Add Docker GPG key
      ansible.builtin.apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      ansible.builtin.apt_repository:
        repo: "deb https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present

    - name: Install Docker
      ansible.builtin.apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-buildx-plugin
          - docker-compose-plugin
        state: present
        update_cache: yes

    - name: Add users to docker group
      ansible.builtin.user:
        name: "{{ item }}"
        groups: docker
        append: yes
      loop: "{{ docker_users }}"

    - name: Configure Docker daemon
      ansible.builtin.copy:
        content: |
          {
            "log-driver": "json-file",
            "log-opts": {
              "max-size": "10m",
              "max-file": "3"
            },
            "storage-driver": "overlay2",
            "live-restore": true
          }
        dest: /etc/docker/daemon.json
        mode: '0644'
      notify: Restart docker

    - name: Ensure Docker is started and enabled
      ansible.builtin.service:
        name: docker
        state: started
        enabled: yes

  handlers:
    - name: Restart docker
      ansible.builtin.service:
        name: docker
        state: restarted
```

### Application Deployment with Rollback

```yaml
---
- name: Deploy Application
  hosts: app_servers
  become: yes
  serial: "50%"
  vars:
    app_name: myapp
    app_user: app
    app_base: "/opt/{{ app_name }}"
    releases_dir: "{{ app_base }}/releases"
    current_link: "{{ app_base }}/current"
    keep_releases: 5

  pre_tasks:
    - name: Get current release (for rollback)
      ansible.builtin.stat:
        path: "{{ current_link }}"
      register: current_release

    - name: Store previous release path
      ansible.builtin.set_fact:
        previous_release: "{{ current_release.stat.lnk_target | default('') }}"
      when: current_release.stat.exists | default(false)

  tasks:
    - name: Deploy new release
      block:
        - name: Create release directory
          ansible.builtin.file:
            path: "{{ releases_dir }}/{{ app_version }}"
            state: directory
            owner: "{{ app_user }}"
            mode: '0755'

        - name: Download and extract application
          ansible.builtin.unarchive:
            src: "https://artifacts.example.com/{{ app_name }}/{{ app_version }}.tar.gz"
            dest: "{{ releases_dir }}/{{ app_version }}"
            remote_src: yes

        - name: Deploy configuration
          ansible.builtin.template:
            src: app-config.yml.j2
            dest: "{{ releases_dir }}/{{ app_version }}/config.yml"
            owner: "{{ app_user }}"
            mode: '0600'

        - name: Update symlink to new release
          ansible.builtin.file:
            src: "{{ releases_dir }}/{{ app_version }}"
            dest: "{{ current_link }}"
            state: link
            force: yes

        - name: Restart application
          ansible.builtin.systemd:
            name: "{{ app_name }}"
            state: restarted

        - name: Wait for application to be healthy
          ansible.builtin.uri:
            url: "http://localhost:8080/health"
            status_code: 200
          register: health
          retries: 10
          delay: 5
          until: health.status == 200

      rescue:
        - name: Rollback - restore previous symlink
          ansible.builtin.file:
            src: "{{ previous_release }}"
            dest: "{{ current_link }}"
            state: link
            force: yes
          when: previous_release | length > 0

        - name: Rollback - restart with previous version
          ansible.builtin.systemd:
            name: "{{ app_name }}"
            state: restarted

        - name: Fail with message
          ansible.builtin.fail:
            msg: "Deployment of {{ app_version }} failed. Rolled back to previous version."

  post_tasks:
    - name: Cleanup old releases
      ansible.builtin.shell: |
        ls -dt {{ releases_dir }}/*/ | tail -n +{{ keep_releases + 1 }} | xargs rm -rf
      changed_when: false
```

---

## 13. Ansible Best Practices

1. **Use FQCN (Fully Qualified Collection Names):** Always use `ansible.builtin.copy` instead of just `copy`
2. **Name every task:** Makes output readable and debugging easier
3. **Use roles for reusability:** Don't put everything in one massive playbook
4. **Keep secrets in Vault:** Never commit plaintext passwords
5. **Use `ansible-lint`:** Catches common mistakes and enforces style
6. **Pin versions:** Pin Ansible, collection, and role versions in CI/CD
7. **Use tags strategically:** Allow partial playbook runs (`--tags`, `--skip-tags`)
8. **Prefer modules over shell/command:** Modules are idempotent; shell commands usually aren't
9. **Use `changed_when: false` for read-only commands:** Prevents false "changed" reports
10. **Use `--check --diff` before applying:** Always dry-run in production
11. **Limit blast radius with `serial`:** Deploy to a percentage of hosts at a time
12. **Use `group_vars/` and `host_vars/` directories:** Keep variables organized
13. **Test with Molecule:** Automated testing framework for roles
14. **Use dynamic inventory in cloud:** Don't manually maintain host lists
15. **Document your roles:** README with variables, dependencies, and examples
16. **Keep playbooks in version control:** Git is your audit trail
17. **Use `block/rescue/always` for critical tasks:** Proper error handling with rollback
18. **Avoid storing large files in roles:** Use artifact repositories or `get_url`
19. **Use `ansible.cfg` per project:** Keeps configuration portable

---

## 14. Common Errors and Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `unreachable` | SSH connection failed | Check SSH key, network, security groups |
| `Permission denied (publickey)` | Wrong SSH key or user | Verify `ansible_user` and key path |
| `Missing sudo password` | Become requires password | Add `-K` flag or configure `NOPASSWD` |
| `MODULE FAILURE` | Module error on target | Check Python version, run with `-vvv` |
| `Variable undefined` | Variable not set | Check spelling, scope, and precedence |
| `Vault password not provided` | Missing vault password | Add `--vault-password-file` or `--ask-vault-pass` |
| `Could not match supplied host pattern` | Host not in inventory | Verify inventory file and host name |
| `Shared connection closed` | SSH timeout/disconnect | Increase timeout, check network stability |
| `AnsibleUndefinedVariable` | Typo or missing variable | Use `{{ var | default('') }}` for optional vars |
| `apt lock` / `dpkg lock` | Another process using apt | Wait or kill stale lock: use `retries` + `delay` |

```yaml
# Handling apt lock with retry
- name: Install packages (with lock retry)
  ansible.builtin.apt:
    name: nginx
    state: present
  register: apt_result
  retries: 5
  delay: 10
  until: apt_result is success
```

---

## 15. Lessons Learned / Pro Tips

1. **Start with `--check --diff`** in production — never blind-apply
2. **Use `serial: 1` for risky changes** — catch issues on one host first
3. **The `command` module is NOT idempotent** — always add `creates:` or `when:` guards
4. **`register` captures everything** — use `.stdout`, `.rc`, `.changed` for conditionals
5. **`ansible_facts` cache** — use `fact_caching = jsonfile` to speed up subsequent runs
6. **`delegate_to: localhost`** — run tasks on control node (API calls, local file operations)
7. **Use `run_once: true`** for tasks that should execute only on one host (DB migrations)
8. **`ansible-playbook --syntax-check`** — fast validation before running
9. **Use `pre_tasks` and `post_tasks`** — ensure ordering around roles
10. **`ansible-config dump --only-changed`** — see non-default configuration
11. **Use `wait_for` module** — wait for ports/files before proceeding
12. **`async` and `poll`** — run long tasks without SSH timeout issues
13. **`throttle: 1`** on tasks — limit parallelism for rate-limited APIs
14. **Test with `--limit one_host` first** — validate before full deployment
15. **Use callback plugins** — `profile_tasks` shows task timing for optimization
16. **`no_log: true`** on tasks with secrets — prevents passwords in output
17. **`environment:` directive** — set env vars for specific tasks (HTTP_PROXY, PATH)

```yaml
# Pro tip: async for long-running tasks
- name: Run long database backup
  ansible.builtin.command: /opt/scripts/full-backup.sh
  async: 3600      # Allow up to 1 hour
  poll: 30         # Check every 30 seconds

# Pro tip: delegate and run_once
- name: Run migration on one host only
  ansible.builtin.command: python manage.py migrate
  args:
    chdir: /opt/app
  run_once: true
  delegate_to: "{{ groups['app_servers'][0] }}"
```

---

## 16. Interview Questions

**Q1: What is Ansible and why is it preferred over other CM tools?**
> Ansible is an agentless configuration management and automation tool that uses SSH and YAML. It's preferred because: no agent installation needed, low learning curve (YAML), large module library, strong community, and push-based model gives immediate feedback.

**Q2: Explain idempotency in Ansible with an example.**
> Idempotency means running a task multiple times produces the same result. Example: `ansible.builtin.apt: name=nginx state=present` — first run installs nginx, subsequent runs detect it's already installed and do nothing (returns "ok" not "changed").

**Q3: What is the difference between `import_tasks` and `include_tasks`?**
> `import_tasks` is static (processed at parse time, supports tags inheritance, no loops). `include_tasks` is dynamic (processed at runtime, supports loops and variable filenames, tags not inherited).

**Q4: How does variable precedence work?**
> Ansible has 22 levels of precedence. Key points: extra vars (-e) always win, role defaults are easiest to override, host_vars beat group_vars, and child group vars beat parent group vars.

**Q5: How do you handle secrets in Ansible?**
> Use Ansible Vault to encrypt sensitive data. You can encrypt entire files (`ansible-vault encrypt`) or individual strings (`ansible-vault encrypt_string`). In CI/CD, store the vault password as a pipeline secret variable.

**Q6: What is the difference between `command` and `shell` modules?**
> `command` executes without a shell (no pipes, redirects, env vars). `shell` uses `/bin/sh` so supports pipes, redirects, and environment variables. `command` is safer (no shell injection risk).

**Q7: Explain handlers and when they trigger.**
> Handlers are tasks that run only when notified by another task that reports "changed". They run once at the end of all tasks (or when `meta: flush_handlers` is called), regardless of how many times they're notified.

**Q8: How do you handle errors in Ansible?**
> Use `ignore_errors`, `failed_when`, `block/rescue/always`, `retries/until`, and `changed_when`. For critical deployments, use block/rescue for rollback logic.

**Q9: What is a dynamic inventory and when would you use it?**
> Dynamic inventory queries a source (AWS, Azure, GCP) at runtime to get current hosts. Use it when infrastructure is dynamic (auto-scaling, cloud environments) so you don't manually maintain host lists.

**Q10: Explain the `serial` keyword.**
> `serial` controls how many hosts are processed at a time (rolling updates). `serial: 1` processes one host at a time. `serial: "25%"` processes 25% of hosts per batch. Essential for zero-downtime deployments.

**Q11: How do you test Ansible roles?**
> Use Molecule — it creates test instances (Docker, Vagrant, cloud), runs your role, then verifies with Ansible assertions or Testinfra. Integrates with CI/CD for automated testing.

**Q12: What are Ansible collections?**
> Collections are the distribution format for Ansible content — modules, plugins, roles, and playbooks packaged together. Installed via `ansible-galaxy collection install`. Examples: `community.docker`, `amazon.aws`.

**Q13: How would you deploy to 100 servers with zero downtime?**
> Use `serial` for rolling updates, health checks after each batch, and `block/rescue` for automatic rollback. Remove servers from load balancer before updating, verify health, then add back.

**Q14: What is `delegate_to` and when do you use it?**
> `delegate_to` runs a task on a different host than the current play target. Common uses: API calls from localhost, removing a host from a load balancer, running commands on a bastion host.

**Q15: How do you optimize Ansible for large environments?**
> Enable pipelining in SSH, increase forks, use fact caching, use `free` strategy, enable Mitogen, use `gather_facts: no` when not needed, use `--limit` for targeted runs.

**Q16: Explain the difference between `vars` and `defaults` in roles.**
> `defaults/main.yml` has the lowest variable precedence — easily overridden by users. `vars/main.yml` has high precedence — harder to override. Use defaults for variables users should customize, vars for internal role constants.

---

## 17. Free Resources

1. **Ansible Official Documentation** — https://docs.ansible.com/
2. **Ansible Galaxy (Roles & Collections)** — https://galaxy.ansible.com/
3. **Jeff Geerling's "Ansible for DevOps" (samples)** — https://github.com/geerlingguy/ansible-for-devops
4. **Ansible Examples Repository** — https://github.com/ansible/ansible-examples
5. **KodeKloud Ansible Free Labs** — https://kodekloud.com/courses/ansible/
6. **Ansible Lint Documentation** — https://ansible.readthedocs.io/projects/lint/
7. **Molecule Testing Framework** — https://ansible.readthedocs.io/projects/molecule/
8. **Red Hat Ansible Automation Blog** — https://www.ansible.com/blog
9. **Learn Linux TV Ansible Series (YouTube)** — https://www.youtube.com/c/LearnLinuxtv
10. **TechWorld with Nana - Ansible Tutorial (YouTube)** — https://www.youtube.com/watch?v=1id6ERvfozo
11. **Ansible Best Practices Guide** — https://docs.ansible.com/ansible/latest/tips_tricks/index.html
12. **Jinja2 Template Designer Docs** — https://jinja.palletsprojects.com/en/3.1.x/templates/

---

*Study Material prepared for DevOps Upskilling Program — GCC Market Focus*
*Last updated: July 2026*
