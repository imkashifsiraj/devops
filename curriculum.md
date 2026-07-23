# DevOps Upskilling Program 

> **Target Audience:** IT professionals looking to transition into or advance in DevOps roles
>
> **Program Duration:** 6 months (part-time) or 3 months (intensive full-time)
>
> **Outcome:** Job-ready DevOps Engineer

---

## Program Structure Overview

```
Month 1: Foundations (Linux, Networking, Git, Scripting)
Month 2: Containerization & Orchestration (Docker, Kubernetes, Helm)
Month 3: Infrastructure as Code & Configuration (Terraform, Ansible)
Month 4: CI/CD Pipelines (GitLab CI, Azure DevOps, Jenkins)
Month 5: Cloud & Monitoring (AWS/Azure, Prometheus, Grafana, ELK)
Month 6: Security, GitOps & Capstone Project
```

---

## Month 1: Foundations

### Week 1-2: Linux Administration

**Learning Objectives:**
- Navigate and manage a Linux system confidently
- Manage users, permissions, and services
- Troubleshoot common system issues

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Linux filesystem hierarchy, basic commands | Navigate a live server, find files, read logs |
| 2 | File permissions, ownership (chmod, chown) | Set up a web server directory with proper permissions |
| 3 | User & group management, sudo | Create users, configure sudoers for a team |
| 4 | Package management (apt/yum), systemd services | Install Nginx, enable/start/troubleshoot the service |
| 5 | Process management, cron jobs | Schedule backup script, monitor processes |
| 6 | Disk management, LVM, mount points | Add a disk, create partitions, set up auto-mount |
| 7 | SSH, key-based auth, SCP/rsync | Set up passwordless SSH between two servers |
| 8 | Log management, journalctl, log rotation | Configure logrotate, analyze auth logs |
| 9 | Performance troubleshooting (top, htop, iostat, vmstat) | Diagnose a slow server scenario |
| 10 | Practical exam: Troubleshoot a broken server | Given 5 issues, fix them within 60 minutes |

**Resources:**
- Practice: KodeKloud Linux Basics labs
- Book: "The Linux Command Line" by William Shotts (free online)

---

### Week 3: Networking Fundamentals

**Learning Objectives:**
- Understand how applications communicate over networks
- Troubleshoot connectivity issues
- Design basic network architectures

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | OSI Model, TCP/UDP, IP addressing, CIDR | Calculate subnets, identify layers for protocols |
| 2 | DNS deep dive (records, resolution, troubleshooting) | Set up a local DNS server, troubleshoot resolution |
| 3 | HTTP/HTTPS, TLS, load balancing concepts | Use curl to inspect requests, set up Nginx as reverse proxy |
| 4 | Firewalls (iptables, security groups), NAT | Configure iptables rules, block/allow specific traffic |
| 5 | Network troubleshooting toolkit (ping, traceroute, dig, ss, tcpdump) | Diagnose 5 network scenarios (port blocked, DNS failure, etc.) |

**Resources:**
- Practice: Build a mini network with Docker containers
- Video: "Networking Fundamentals" by NetworkChuck

---

### Week 4: Git & Shell Scripting

**Learning Objectives:**
- Use Git professionally (branching, PRs, conflicts)
- Write production-quality Bash scripts
- Automate repetitive tasks

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Git fundamentals (init, add, commit, push, pull) | Create repo, make commits, push to GitLab/GitHub |
| 2 | Branching, merging, conflict resolution | Simulate merge conflict, resolve it properly |
| 3 | Git flow, pull requests, code review | Create feature branch, submit MR, review peer's code |
| 4 | Bash scripting: variables, conditionals, loops | Write a server health check script |
| 5 | Bash scripting: functions, error handling, practical scripts | Write automated backup script with notifications |
| 6 | Python basics for DevOps (requests, boto3, YAML parsing) | Write script to interact with an API, parse JSON |

**Mini Project:** Build an automated server provisioning script that:
- Creates a user with SSH key
- Installs required packages
- Configures a web server
- Sets up a cron job for log cleanup

---

## Month 2: Containerization & Orchestration

### Week 5-6: Docker

**Learning Objectives:**
- Containerize any application
- Write production-grade Dockerfiles
- Manage multi-container applications with Docker Compose

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Containers vs VMs, Docker architecture | Install Docker, run first containers, explore |
| 2 | Docker images, Dockerfile basics | Containerize a Node.js/Python app |
| 3 | Multi-stage builds, image optimization | Reduce image from 800MB to 80MB |
| 4 | Docker networking (bridge, host, custom networks) | Connect app container to database container |
| 5 | Docker volumes, persistent data | Set up PostgreSQL with persistent storage |
| 6 | Docker Compose for multi-service apps | Build full stack: app + DB + Redis + Nginx |
| 7 | Container security (non-root, scanning, best practices) | Scan images with Trivy, fix vulnerabilities |
| 8 | Docker registry (push/pull, private registry) | Push to Docker Hub and private registry |
| 9 | Troubleshooting containers (logs, exec, inspect) | Debug 5 broken container scenarios |
| 10 | Practical exam: Containerize a full application stack | Given source code, produce production-ready containers |

**Mini Project:** Containerize a 3-tier application:
- Frontend (React/Nginx)
- Backend API (Node.js/Python)
- Database (PostgreSQL)
- Include: health checks, non-root user, optimized images, compose file

---

### Week 7-8: Kubernetes

**Learning Objectives:**
- Deploy and manage applications on Kubernetes
- Troubleshoot common K8s issues
- Understand K8s networking and storage

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | K8s architecture (control plane, nodes, components) | Set up minikube/kind, explore cluster |
| 2 | Pods, ReplicaSets, Deployments | Deploy app, scale up/down, observe behavior |
| 3 | Services (ClusterIP, NodePort, LoadBalancer) | Expose app internally and externally |
| 4 | ConfigMaps, Secrets, environment management | Configure app with different envs (dev/prod) |
| 5 | Persistent Volumes, PVCs, StorageClasses | Deploy PostgreSQL with persistent storage |
| 6 | Ingress controllers, TLS termination | Set up Nginx Ingress with path-based routing |
| 7 | Health probes (liveness, readiness, startup) | Configure probes, observe pod lifecycle |
| 8 | Resource management (requests, limits, HPA) | Set up autoscaling, trigger with load test |
| 9 | Namespaces, RBAC, NetworkPolicies | Multi-tenant setup with isolation |
| 10 | Troubleshooting (pending pods, crashloop, OOMKilled) | Diagnose and fix 10 broken deployments |
| 11 | DaemonSets, StatefulSets, Jobs/CronJobs | Deploy log collector, stateful database |
| 12 | Helm basics (install charts, custom values) | Deploy Redis/Prometheus via Helm |

**Mini Project:** Deploy a production-like application on K8s:
- Multi-replica deployment with HPA
- Ingress with TLS
- ConfigMaps for configuration
- Persistent storage for database
- Monitoring with Prometheus (via Helm)

**Certification Prep:** Begin CKA (Certified Kubernetes Administrator) preparation

---

## Month 3: Infrastructure as Code & Configuration Management

### Week 9-10: Terraform

**Learning Objectives:**
- Provision cloud infrastructure using code
- Manage state, modules, and workspaces
- Implement Terraform in CI/CD pipelines

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | IaC concepts, Terraform architecture, providers | Install Terraform, provision first resource |
| 2 | HCL syntax, resources, variables, outputs | Create VPC with subnets on AWS/Azure |
| 3 | State management (local vs remote, locking) | Set up S3/Azure Blob backend with locking |
| 4 | Data sources, dependencies, lifecycle rules | Reference existing resources, manage replacement |
| 5 | Modules (using community modules) | Deploy VPC using terraform-aws-modules |
| 6 | Writing custom modules | Create reusable module for web app deployment |
| 7 | Workspaces and environment management | Manage dev/staging/prod with same code |
| 8 | Terraform + CI/CD (plan on PR, apply on merge) | Set up GitLab CI pipeline for Terraform |
| 9 | Import existing infrastructure, state manipulation | Import manually-created resources |
| 10 | Best practices, code organization, security scanning | Structure a real project, run tfsec/checkov |

**Mini Project:** Build complete cloud infrastructure:
- VPC with public/private subnets across 2 AZs
- EKS/AKS cluster
- RDS/managed database
- Load balancer
- All via Terraform modules with remote state

**Certification Prep:** HashiCorp Terraform Associate

---

### Week 11-12: Ansible

**Learning Objectives:**
- Configure servers at scale
- Write reusable roles and playbooks
- Integrate Ansible with CI/CD

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Ansible architecture (agentless, SSH, inventory) | Set up inventory, run ad-hoc commands |
| 2 | Playbooks basics (tasks, modules, handlers) | Write playbook to install and configure Nginx |
| 3 | Variables, facts, conditionals, loops | Configure different services based on OS |
| 4 | Templates (Jinja2), file management | Template Nginx config with dynamic values |
| 5 | Roles (structure, defaults, dependencies) | Refactor playbook into reusable role |
| 6 | Ansible Vault (encrypting secrets) | Encrypt database passwords, use in playbook |
| 7 | Dynamic inventory (AWS/Azure integration) | Auto-discover EC2/VM instances |
| 8 | Ansible + Terraform (provision then configure) | Terraform creates VMs, Ansible configures them |
| 9 | Ansible in CI/CD pipelines | Trigger Ansible from GitLab CI on merge |
| 10 | Practical exam: Configure a fleet of servers | Given requirements, write complete Ansible automation |

**Mini Project:** Automate complete server fleet configuration:
- Common role (users, SSH, security hardening)
- Web server role (Nginx, TLS, virtual hosts)
- Monitoring agent role (node_exporter)
- All encrypted with Vault, triggered via CI/CD

---

## Month 4: CI/CD Pipelines

### Week 13-14: Azure DevOps

**Learning Objectives:**
- Build complete CI/CD pipelines in Azure DevOps
- Manage repositories, boards, and artifacts
- Deploy to Kubernetes and cloud services

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Azure DevOps overview (Repos, Boards, Pipelines, Artifacts) | Set up organization, project, repo |
| 2 | YAML pipelines (triggers, stages, jobs, steps) | Create build pipeline for Docker image |
| 3 | Multi-stage pipelines (build -> test -> deploy) | Full pipeline with staging and production |
| 4 | Environments, approvals, deployment gates | Configure production approval workflow |
| 5 | Variable groups, key vault integration, secrets | Secure pipeline with Azure Key Vault |
| 6 | Artifacts, feeds (npm, NuGet, Docker) | Publish and consume packages |
| 7 | Deploy to AKS (Kubernetes) | Pipeline deploys to Azure Kubernetes Service |
| 8 | Infrastructure pipelines (Terraform via Azure DevOps) | IaC pipeline with plan/apply stages |
| 9 | Templates and reusable pipeline libraries | Create organization-wide pipeline templates |
| 10 | Self-hosted agents, agent pools | Set up and configure self-hosted agents |

---

### Week 15: GitLab CI/CD

**Learning Objectives:**
- Build CI/CD pipelines in GitLab
- Use advanced features (DAG, multi-project, review apps)

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | .gitlab-ci.yml structure, stages, jobs | Create basic pipeline (build, test, deploy) |
| 2 | Variables, caching, artifacts | Optimize pipeline speed |
| 3 | Docker-in-Docker, container registry | Build and push images in pipeline |
| 4 | Environments, review apps, manual gates | Deploy preview per merge request |
| 5 | Advanced: includes, templates, child pipelines | Organization-wide CI templates |

---

### Week 16: Jenkins (Legacy Knowledge)

**Learning Objectives:**
- Understand Jenkins architecture and pipelines
- Maintain and improve existing Jenkins setups

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Jenkins architecture (controller, agents, plugins) | Install Jenkins, configure agent |
| 2 | Declarative pipelines (Jenkinsfile) | Write multi-stage pipeline |
| 3 | Shared libraries, credentials, Blue Ocean | Create shared library for team |
| 4 | Jenkins + Kubernetes (dynamic agents) | Configure K8s plugin for ephemeral agents |
| 5 | Migration strategies (Jenkins -> GitLab/Azure DevOps) | Plan migration of real pipelines |

---

## Month 5: Cloud & Monitoring

### Week 17-18: Cloud Platform (AWS or Azure)

**Choose based on target sector:**
- **Azure** → Government, banking, enterprise
- **AWS** → Startups, tech companies, international firms

**Azure Track Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Azure fundamentals (regions, resource groups, subscriptions) | Create resource group, explore portal |
| 2 | Networking (VNet, subnets, NSGs, Azure Firewall) | Build hub-spoke network architecture |
| 3 | Compute (VMs, VM Scale Sets, App Service) | Deploy and scale application |
| 4 | AKS (Azure Kubernetes Service) | Deploy managed K8s cluster |
| 5 | Storage (Blob, Disks, Files) | Configure storage for applications |
| 6 | Azure AD (Entra ID), RBAC, Managed Identities | Set up service principals, workload identity |
| 7 | Azure Monitor, Log Analytics, Application Insights | Configure monitoring for AKS workload |
| 8 | Azure Key Vault, Defender for Cloud | Secure infrastructure and secrets |
| 9 | Azure DevOps + Azure integration (service connections) | End-to-end deployment pipeline |
| 10 | Cost management, tagging, governance (Azure Policy) | Implement tagging policy, budget alerts |

**AWS Track Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | AWS fundamentals (regions, AZs, IAM) | Configure IAM users, roles, policies |
| 2 | VPC (subnets, route tables, NAT, security groups) | Build production VPC architecture |
| 3 | EC2, Auto Scaling Groups, ALB | Deploy auto-scaling web tier |
| 4 | EKS (Elastic Kubernetes Service) | Deploy managed K8s cluster |
| 5 | S3, EBS, EFS | Configure storage solutions |
| 6 | RDS, ElastiCache, DynamoDB | Deploy managed databases |
| 7 | CloudWatch, CloudTrail, X-Ray | Set up monitoring and auditing |
| 8 | Secrets Manager, KMS, WAF | Secure application and data |
| 9 | ECS/Fargate (container alternatives to K8s) | Deploy serverless containers |
| 10 | Cost Explorer, Trusted Advisor, Well-Architected | Optimize and review architecture |

**Certification Prep:** AWS Solutions Architect Associate OR Azure Administrator (AZ-104)

---

### Week 19-20: Monitoring & Observability

**Learning Objectives:**
- Implement complete observability stack
- Write effective alerts
- Build useful dashboards

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | Observability concepts (metrics, logs, traces) | Architecture overview and planning |
| 2 | Prometheus (installation, configuration, scraping) | Deploy Prometheus, scrape app metrics |
| 3 | PromQL queries (rate, histogram, aggregations) | Write queries for error rate, latency, saturation |
| 4 | Grafana dashboards (panels, variables, templates) | Build service dashboard with RED metrics |
| 5 | Alerting (Alertmanager, routing, silencing) | Configure alerts for SLO violations |
| 6 | Logging: EFK stack or Grafana Loki | Deploy centralized logging, search across services |
| 7 | Distributed tracing (Jaeger or Tempo) | Instrument app, trace requests across services |
| 8 | Kubernetes monitoring (kube-prometheus-stack) | Deploy full monitoring stack via Helm |
| 9 | On-call, incident response, runbooks | Write runbooks for common scenarios |
| 10 | SLIs, SLOs, error budgets (SRE practices) | Define and monitor SLOs for application |

**Mini Project:** Complete observability stack:
- Prometheus + Grafana for metrics
- Loki for logs
- Alertmanager with Slack/Teams notifications
- Dashboards for infrastructure AND application
- Runbook for top 5 alerts

---

## Month 6: Security, GitOps & Capstone

### Week 21-22: DevSecOps & Security

**Learning Objectives:**
- Integrate security into CI/CD pipelines
- Secure Kubernetes workloads
- Understand compliance requirements relevant to target market

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | DevSecOps principles, shift-left security | Add security gates to existing pipeline |
| 2 | SAST (SonarQube, Semgrep) | Integrate code scanning in CI |
| 3 | Dependency scanning (Snyk, Trivy) | Find and fix vulnerable dependencies |
| 4 | Container image scanning and signing | Scan images in pipeline, block critical CVEs |
| 5 | Kubernetes security (RBAC, PSA, NetworkPolicies) | Harden a cluster with least-privilege |
| 6 | Infrastructure scanning (Checkov, tfsec) | Scan Terraform before apply |
| 7 | Secret management (Vault, External Secrets Operator) | Integrate Vault with Kubernetes |
| 8 | Supply chain security (SBOM, image signing) | Generate SBOM, sign with cosign |
| 9 | Market compliance overview (SAMA, NESA, PDPL) | Map security controls to compliance requirements |
| 10 | Security incident response and forensics | Simulate and respond to a security event |

**Market-Specific Compliance:**
- **SAMA** — Banking/fintech regulations
- **NESA** — Government security standards
- **PDPL** — Data privacy
- **NIA** — National Information Assurance framework

---

### Week 23: GitOps with ArgoCD

**Learning Objectives:**
- Implement GitOps workflow
- Deploy and manage ArgoCD
- Handle multi-environment deployments

**Topics:**
| Day | Topic | Hands-On Lab |
|-----|-------|--------------|
| 1 | GitOps principles, ArgoCD architecture | Install ArgoCD, deploy first application |
| 2 | Application CRDs, sync policies, self-healing | Configure auto-sync, test drift correction |
| 3 | Kustomize overlays for multi-environment | Manage dev/staging/prod from one repo |
| 4 | ArgoCD + Helm, ApplicationSets | Deploy fleet of applications |
| 5 | Secrets in GitOps (Sealed Secrets, External Secrets) | Encrypted secrets committed to Git |

---

### Week 24: Capstone Project & Interview Prep

**Capstone Project — Build Production-Grade Infrastructure:**

Students build an end-to-end platform simulating a real enterprise project:

```
Requirements:
1. Infrastructure (Terraform):
   - VPC/VNet with public/private subnets
   - Managed Kubernetes cluster (EKS or AKS)
   - Managed database (RDS or Azure SQL)
   - Container registry

2. Application Deployment (Kubernetes):
   - 3-tier app (frontend + API + database)
   - Helm charts with per-environment values
   - HPA for auto-scaling
   - Ingress with TLS
   - Health probes and resource limits

3. CI/CD Pipeline (Azure DevOps or GitLab):
   - Build and test on merge request
   - Security scanning (SAST + container scan)
   - Build and push Docker image
   - Deploy to staging (automatic)
   - Deploy to production (manual approval)

4. GitOps (ArgoCD):
   - Separate config repository
   - ArgoCD manages K8s deployments
   - Self-healing enabled

5. Monitoring:
   - Prometheus + Grafana
   - Application dashboards
   - Alerts for SLO violations
   - Centralized logging

6. Security:
   - Non-root containers
   - NetworkPolicies
   - Secrets from external secret store
   - RBAC configured
   - Image scanning in pipeline

7. Documentation:
   - Architecture diagram
   - Runbooks for common issues
   - ADR (Architecture Decision Records)
```

**Interview Preparation:**
| Day | Activity |
|-----|----------|
| 1 | Resume workshop (DevOps-focused resume for job market) |
| 2 | Technical interview practice (system design, troubleshooting) |
| 3 | Behavioral interview practice (STAR method, incident stories) |
| 4 | Mock interviews (panel format, common in job market) |
| 5 | Final presentation: Present capstone project |

---

## Certification Roadmap 

```
Order of importance for intended job market:

1. CKA (Certified Kubernetes Administrator)
   - Most requested certification in Job market DevOps roles
   - Hands-on exam (practical, not multiple choice)
   - Prep time: 4-6 weeks

2. Azure Administrator (AZ-104) OR AWS Solutions Architect Associate
   - Choose based on target employer
   - Azure for government/enterprise
   - AWS for tech companies/startups
   - Prep time: 4-6 weeks

3. HashiCorp Terraform Associate
   - Validates IaC skills
   - Multiple choice (easier than CKA)
   - Prep time: 2-3 weeks

4. Optional Advanced:
   - CKS (Certified Kubernetes Security Specialist)
   - AWS DevOps Engineer Professional
   - Azure DevOps Engineer Expert (AZ-400)
```

---

## Program Delivery Format

### Recommended Structure
```
Live Sessions (3x per week, 2 hours each):
  - Theory + Demo (30 min)
  - Guided Lab (60 min)
  - Q&A / Review (30 min)

Self-Paced Labs (daily):
  - 1-2 hours hands-on practice
  - KodeKloud / Cloud Playground access
  - Assignments with deadlines

Weekly:
  - Office hours (1 hour, open questions)
  - Peer review (review each other's code/pipelines)
  - Weekly quiz (verify understanding)

Monthly:
  - Mini project presentation
  - Progress assessment
  - 1-on-1 mentoring session
```

### Lab Environment Options
```
Option A:
  - Docker Desktop / Minikube (local)
  - AWS/Azure free tier
  - GitLab.com free tier
  - GitHub free tier

Option B (Recommended):
  - KodeKloud subscription ($15/month)
  - Cloud sandbox (A Cloud Guru or Whizlabs)
  - Shared Azure DevOps organization
  - Shared Kubernetes cluster

Option C (Enterprise):
  - Dedicated cloud accounts per student
  - Enterprise GitLab/Azure DevOps
  - Dedicated lab environment
  - Exam vouchers included
```

---

## Assessment & Graduation Criteria

```
To graduate, students must:

[ ] Pass Linux administration practical exam (70%+)
[ ] Containerize an application with Docker (graded)
[ ] Deploy application on Kubernetes (graded)
[ ] Write Terraform for cloud infrastructure (graded)
[ ] Build complete CI/CD pipeline (graded)
[ ] Implement monitoring stack (graded)
[ ] Complete capstone project (graded by panel)
[ ] Present capstone project (soft skills assessment)
[ ] Obtain at least 1 certification (CKA recommended)
[ ] Contribute to peer code reviews (participation)
```

---

## Job Placement Support

```
Resume:
  - DevOps-specific resume template
  - GitHub/GitLab profile optimization
  - LinkedIn profile review

Portfolio:
  - Capstone project on GitHub (public)
  - Blog posts documenting learnings
  - Certifications linked

Interview Prep:
  - Mock technical interviews
  - Whiteboard architecture practice
  - Salary negotiation for Job market

---


> **Key Differentiator for Your Program:** Focus on hands-on labs (80% practical, 20% theory), Job market-specific compliance knowledge (SAMA, NESA), Azure DevOps emphasis, and interview preparation tailored to hiring patterns (panel interviews, certification verification, practical assessments).
