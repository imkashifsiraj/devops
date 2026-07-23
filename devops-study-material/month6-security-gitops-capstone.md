# Month 6: Security, GitOps & Capstone Project

## DevOps Upskilling Program — GCC Market Focus

**Duration:** Month 6 of 6  
**Prerequisites:** Months 1-5 (Linux, Cloud, IaC, CI/CD, Containers, Kubernetes, Monitoring)  
**Objective:** Master DevSecOps, GitOps with ArgoCD, and deliver a capstone project

---

# SECTION 1: DEVSECOPS & SECURITY

---

## 1.1 DevSecOps Concepts

### What is DevSecOps?

DevSecOps integrates security practices within the DevOps process. Security is **everyone's responsibility** — not just the security team's. It embeds security at every phase of the software development lifecycle.

**Traditional approach:**
```
Dev → Ops → Security (at the end, finding issues too late)
```

**DevSecOps approach:**
```
Plan(Sec) → Code(Sec) → Build(Sec) → Test(Sec) → Deploy(Sec) → Operate(Sec) → Monitor(Sec)
```

**Key principles:**
- Security is a shared responsibility across all teams
- Automate security checks wherever possible
- Security should not slow down delivery — it should be built into the flow
- Continuous security monitoring and improvement
- Culture of security awareness

### Shift-Left Security

"Shift-left" means moving security earlier in the development lifecycle.

```
Traditional:    Plan → Code → Build → Test → Deploy → [SECURITY CHECK] → Production
Shift-Left:     Plan → [SEC] → Code → [SEC] → Build → [SEC] → Test → [SEC] → Deploy → [SEC]
```

**Why shift left?**
- Cost of fixing bugs increases 6x-100x as they move through stages
- A vulnerability found in production costs ~30x more than one found in design
- Earlier detection = faster remediation = less risk

**Practical shift-left activities:**
| Phase | Security Activity |
|-------|-------------------|
| Plan | Threat modeling, security requirements |
| Code | IDE security plugins, pre-commit hooks, peer review |
| Build | SAST, dependency scanning, container scanning |
| Test | DAST, penetration testing, fuzzing |
| Deploy | IaC scanning, config validation |
| Operate | Runtime protection, monitoring, incident response |

### Security in the SDLC

```
┌─────────────────────────────────────────────────────────┐
│                  Secure SDLC                             │
├──────────┬──────────┬──────────┬──────────┬────────────┤
│ REQUIRE  │  DESIGN  │  DEVELOP │   TEST   │  DEPLOY    │
├──────────┼──────────┼──────────┼──────────┼────────────┤
│ Security │ Threat   │ Secure   │ SAST/    │ Config     │
│ require- │ modeling │ coding   │ DAST     │ hardening  │
│ ments    │          │ standards│          │            │
│          │ Attack   │ Code     │ Pen test │ Runtime    │
│ Abuse    │ surface  │ review   │          │ protection │
│ cases    │ analysis │          │ Fuzzing  │            │
│          │          │ IDE      │          │ Monitoring │
│ Risk     │ Security │ plugins  │ SCA      │            │
│ assess.  │ patterns │          │          │ Incident   │
│          │          │          │          │ response   │
└──────────┴──────────┴──────────┴──────────┴────────────┘
```

### Defense in Depth

Multiple layers of security controls throughout an IT system. If one layer fails, the next catches the threat.

```
┌─────────────────────────────────────────┐
│           Layer 7: Data                 │  Encryption, DLP, classification
├─────────────────────────────────────────┤
│           Layer 6: Application          │  WAF, input validation, SAST/DAST
├─────────────────────────────────────────┤
│           Layer 5: Host/Container       │  Hardening, antivirus, patching
├─────────────────────────────────────────┤
│           Layer 4: Internal Network     │  Segmentation, Network Policies
├─────────────────────────────────────────┤
│           Layer 3: Perimeter            │  Firewalls, IDS/IPS, DDoS protection
├─────────────────────────────────────────┤
│           Layer 2: Identity & Access    │  MFA, RBAC, SSO
├─────────────────────────────────────────┤
│           Layer 1: Physical             │  Data center security, hardware
└─────────────────────────────────────────┘
```

### Zero Trust Architecture Principles

**Core mantra:** "Never trust, always verify"

**Principles:**
1. **Verify explicitly** — Always authenticate and authorize based on all available data points
2. **Least privilege access** — Limit access with just-in-time and just-enough-access (JIT/JEA)
3. **Assume breach** — Minimize blast radius, segment access, verify end-to-end encryption

**Zero Trust in DevOps context:**
```yaml
# Example: Zero Trust network policy in Kubernetes
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-default
  namespace: production
spec:
  podSelector: {}          # Apply to all pods
  policyTypes:
    - Ingress
    - Egress
  ingress: []              # Deny all ingress by default
  egress: []               # Deny all egress by default
```

**Implementation areas:**
- Network: Micro-segmentation, mTLS between services
- Identity: Strong authentication, short-lived tokens
- Devices: Device compliance, health checks
- Applications: Runtime verification, API security
- Data: Encryption at rest and in transit, classification

### Principle of Least Privilege

Every user, process, and system should have only the minimum permissions needed to perform its function.

**Examples in DevOps:**
```yaml
# BAD: Over-privileged service account
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: app-admin
subjects:
  - kind: ServiceAccount
    name: my-app
roleRef:
  kind: ClusterRole
  name: cluster-admin    # NEVER do this for applications

---
# GOOD: Minimal permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
  namespace: my-app-ns
rules:
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["get", "list"]
    resourceNames: ["my-app-config"]  # Even more restrictive
```

### CIA Triad

| Principle | Definition | DevOps Example |
|-----------|-----------|----------------|
| **Confidentiality** | Data accessible only to authorized parties | Encryption, secret management, RBAC |
| **Integrity** | Data not modified by unauthorized parties | Image signing, checksums, audit logs |
| **Availability** | Systems accessible when needed | HA deployments, auto-scaling, DR |

---

## 1.2 Security in CI/CD Pipeline

### Pre-commit Hooks

Catch issues before code even reaches the repository.

```bash
# Install pre-commit framework
pip install pre-commit

# Create .pre-commit-config.yaml
cat > .pre-commit-config.yaml << 'EOF'
repos:
  # Detect secrets before they're committed
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  # Security linting for Python
  - repo: https://github.com/PyCQA/bandit
    rev: '1.7.5'
    hooks:
      - id: bandit
        args: ['-r', '--severity-level', 'medium']

  # Terraform security
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.77.0
    hooks:
      - id: terraform_tfsec

  # General security
  - repo: https://github.com/zricethezav/gitleaks
    rev: v8.16.1
    hooks:
      - id: gitleaks
EOF

# Install the hooks
pre-commit install

# Run against all files
pre-commit run --all-files
```

### SAST (Static Application Security Testing)

SAST analyzes source code **without executing it** to find security vulnerabilities.

**Key tools:**
| Tool | Language | Type |
|------|----------|------|
| SonarQube | Multi-language | Commercial/Community |
| Semgrep | Multi-language | Open Source |
| CodeQL | Multi-language | Free for OSS (GitHub) |
| Bandit | Python | Open Source |
| Gosec | Go | Open Source |
| ESLint Security | JavaScript | Open Source |

**GitLab CI SAST Example:**
```yaml
# .gitlab-ci.yml
stages:
  - test
  - security

include:
  - template: Security/SAST.gitlab-ci.yml

sast:
  stage: security
  variables:
    SAST_EXCLUDED_ANALYZERS: "eslint"
    SCAN_KUBERNETES_MANIFESTS: "true"

# Custom SAST with Semgrep
semgrep-scan:
  stage: security
  image: returntocorp/semgrep:latest
  script:
    - semgrep scan --config=auto --json --output=semgrep-results.json .
    - semgrep scan --config=p/owasp-top-ten .
    - semgrep scan --config=p/security-audit .
  artifacts:
    reports:
      sast: semgrep-results.json
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

**Azure DevOps SAST Integration:**
```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: SonarQubePrepare@5
    inputs:
      SonarQube: 'SonarQubeConnection'
      scannerMode: 'CLI'
      configMode: 'manual'
      cliProjectKey: 'my-project'
      cliSources: 'src'
      extraProperties: |
        sonar.security.hotspots.reviewed=true
        sonar.qualitygate.wait=true

  - script: |
      npm install
      npm run build
    displayName: 'Build Application'

  - task: SonarQubeAnalyze@5
    displayName: 'Run SAST Analysis'

  - task: SonarQubePublish@5
    inputs:
      pollingTimeoutSec: '300'

  # Microsoft Security DevOps (includes multiple tools)
  - task: MicrosoftSecurityDevOps@1
    displayName: 'Microsoft Security DevOps'
    inputs:
      categories: 'SAST'
```

### DAST (Dynamic Application Security Testing)

DAST tests a **running application** by simulating attacks against it.

**Key tools:**
- **OWASP ZAP** — Free, open-source, most popular
- **Burp Suite** — Commercial, very powerful
- **Nuclei** — Fast, template-based scanner

**Pipeline Integration Example:**
```yaml
# GitLab CI DAST with OWASP ZAP
dast:
  stage: security
  image: ghcr.io/zaproxy/zaproxy:stable
  variables:
    DAST_TARGET_URL: "https://staging.myapp.example.com"
  script:
    - mkdir -p /zap/wrk
    # Baseline scan (passive, fast)
    - zap-baseline.py -t $DAST_TARGET_URL -r zap-report.html -J zap-report.json
    # Full scan (active, thorough — use only in staging)
    # - zap-full-scan.py -t $DAST_TARGET_URL -r zap-full-report.html
  artifacts:
    paths:
      - zap-report.html
      - zap-report.json
    reports:
      dast: zap-report.json
  only:
    - main
  environment:
    name: staging
    url: $DAST_TARGET_URL
```

### Dependency/SCA Scanning

Software Composition Analysis identifies vulnerabilities in third-party dependencies.

**Why it matters:** ~80% of modern application code comes from open-source libraries.

**Tools:**
| Tool | Ecosystem | Integration |
|------|-----------|-------------|
| Snyk | Multi | CI/CD, IDE, Git |
| Dependabot | Multi | GitHub native |
| OWASP Dependency-Check | Java, .NET | CI/CD |
| npm audit | Node.js | CLI, CI/CD |
| pip-audit | Python | CLI, CI/CD |
| Trivy | Multi | CLI, CI/CD |

**Pipeline Example:**
```yaml
# GitLab CI dependency scanning
dependency-scan:
  stage: security
  image: node:18
  script:
    - npm audit --json > npm-audit.json || true
    - npx audit-ci --critical
  artifacts:
    paths:
      - npm-audit.json
  allow_failure:
    exit_codes: 1

# Using Snyk
snyk-scan:
  stage: security
  image: snyk/snyk:node
  script:
    - snyk auth $SNYK_TOKEN
    - snyk test --severity-threshold=high --json > snyk-results.json
    - snyk monitor  # Upload to Snyk dashboard
  artifacts:
    paths:
      - snyk-results.json
```

### Container Image Scanning

Scans container images for known vulnerabilities (CVEs) in OS packages and application dependencies.

**Tools comparison:**
| Tool | Speed | Database | Integration |
|------|-------|----------|-------------|
| Trivy | Fast | Multiple (NVD, Red Hat, etc.) | Excellent |
| Grype | Fast | Anchore DB | Good |
| Docker Scout | Medium | Docker DB | Docker native |
| Snyk Container | Medium | Snyk DB | Excellent |

**GitLab CI Example:**
```yaml
container-scan:
  stage: security
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  variables:
    IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - trivy image --exit-code 0 --severity LOW,MEDIUM --format json -o trivy-low.json $IMAGE
    - trivy image --exit-code 1 --severity HIGH,CRITICAL --format json -o trivy-high.json $IMAGE
  artifacts:
    paths:
      - trivy-low.json
      - trivy-high.json
    reports:
      container_scanning: trivy-high.json
```

**Azure DevOps Example:**
```yaml
- task: Docker@2
  inputs:
    command: build
    repository: myapp
    tags: $(Build.BuildId)

- script: |
    # Install Trivy
    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
    # Scan the built image
    trivy image --exit-code 1 --severity HIGH,CRITICAL \
      --format template --template "@contrib/junit.tpl" \
      -o trivy-results.xml \
      myapp:$(Build.BuildId)
  displayName: 'Container Image Scan'

- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: 'trivy-results.xml'
  condition: always()
```

**Interpreting and fixing CVEs:**
```bash
# Example Trivy output
# CVE-2023-12345 | HIGH | libssl | 1.1.1k-1 | Fixed in 1.1.1l-1
# Fix: Update base image or specific package

# In Dockerfile — fix by updating base image
FROM node:18.17.0-alpine3.18  # Pin to patched version

# Or update specific package
RUN apk update && apk upgrade libssl libcrypto
```

### Infrastructure as Code Scanning

Detect misconfigurations in IaC files before deployment.

**Tools:**
| Tool | IaC Support | Features |
|------|-------------|----------|
| Checkov | TF, K8s, ARM, CloudFormation | 1000+ policies |
| tfsec | Terraform | Fast, focused |
| KICS | Multi | Broad support |
| Terrascan | Multi | OPA-based policies |

**Pipeline Example for Terraform:**
```yaml
# GitLab CI
iac-security:
  stage: security
  image:
    name: bridgecrew/checkov:latest
    entrypoint: [""]
  script:
    - checkov -d ./terraform/ --output json --output-file checkov-results.json
    - checkov -d ./terraform/ --check CKV_AWS_18,CKV_AWS_21  # Specific checks
    - checkov -d ./kubernetes/ --framework kubernetes
  artifacts:
    paths:
      - checkov-results.json

# Azure DevOps
- script: |
    pip install checkov
    checkov -d ./terraform/ \
      --soft-fail \
      --output junitxml > checkov-results.xml
  displayName: 'IaC Security Scan'
```

### License Compliance Scanning

Ensures dependencies don't use licenses incompatible with your project.

```yaml
license-scan:
  stage: security
  image: node:18
  script:
    - npx license-checker --production --json > licenses.json
    - npx license-checker --production --failOn "GPL-3.0;AGPL-3.0"
  artifacts:
    paths:
      - licenses.json
```

### SBOM (Software Bill of Materials)

An SBOM is a complete inventory of all components in your software — think of it as a "nutrition label" for software.

**Why SBOM matters:**
- Required by US Executive Order 14028 for government software
- Enables rapid vulnerability response (e.g., Log4Shell)
- Supply chain transparency
- GCC governments increasingly requiring SBOMs

**Generating SBOM with Syft:**
```bash
# Generate SBOM from container image
syft myapp:latest -o spdx-json > sbom.spdx.json
syft myapp:latest -o cyclonedx-json > sbom.cdx.json

# Generate SBOM from directory
syft dir:./src -o spdx-json > sbom-source.json

# Scan SBOM for vulnerabilities with Grype
grype sbom:sbom.cdx.json
```

### Image Signing and Verification (Cosign)

Ensures container images haven't been tampered with.

```bash
# Generate key pair
cosign generate-key-pair

# Sign an image
cosign sign --key cosign.key myregistry.io/myapp:v1.0

# Verify an image
cosign verify --key cosign.pub myregistry.io/myapp:v1.0

# Keyless signing with Sigstore (recommended)
cosign sign myregistry.io/myapp:v1.0  # Uses OIDC identity

# Verify keyless signature
cosign verify --certificate-identity=user@company.com \
  --certificate-oidc-issuer=https://accounts.google.com \
  myregistry.io/myapp:v1.0

# Attach SBOM to image
cosign attach sbom --sbom sbom.cdx.json myregistry.io/myapp:v1.0
```


---

## 1.3 Container Security

### Minimal Base Images

| Image Type | Size | Attack Surface | Use Case |
|-----------|------|----------------|----------|
| `scratch` | 0 MB | Minimal | Static Go binaries |
| `distroless` | ~20 MB | Very low | Java, Python, Node.js |
| `alpine` | ~5 MB | Low | General purpose |
| `ubuntu` | ~77 MB | Medium | Development, full tooling |
| `ubuntu:full` | ~200 MB | High | Avoid in production |

### Non-Root Containers

```dockerfile
# INSECURE: Running as root (default)
FROM node:18
COPY . /app
CMD ["node", "app.js"]  # Runs as root!

# SECURE: Running as non-root
FROM node:18-alpine
RUN addgroup -g 1001 appgroup && \
    adduser -u 1001 -G appgroup -D appuser
WORKDIR /app
COPY --chown=appuser:appgroup . .
USER appuser
EXPOSE 3000
CMD ["node", "app.js"]
```

### Read-Only Filesystem

```yaml
# Kubernetes: Read-only root filesystem
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
spec:
  template:
    spec:
      containers:
        - name: app
          image: myapp:v1.0
          securityContext:
            readOnlyRootFilesystem: true
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /app/cache
      volumes:
        - name: tmp
          emptyDir: {}
        - name: cache
          emptyDir:
            sizeLimit: 100Mi
```

### Drop All Capabilities

```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE  # Only if binding to port < 1024
```

### Complete Secure Dockerfile

```dockerfile
# === Build Stage ===
FROM node:18-alpine AS builder
WORKDIR /build
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
RUN npm run build

# === Production Stage ===
FROM gcr.io/distroless/nodejs18-debian12:nonroot

# Labels for identification
LABEL maintainer="devops@company.com"
LABEL version="1.0"
LABEL org.opencontainers.image.source="https://github.com/company/app"

# Copy only production artifacts
COPY --from=builder --chown=nonroot:nonroot /build/dist /app/dist
COPY --from=builder --chown=nonroot:nonroot /build/node_modules /app/node_modules
COPY --from=builder --chown=nonroot:nonroot /build/package.json /app/

WORKDIR /app
USER nonroot
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD ["node", "-e", "require('http').get('http://localhost:3000/health')"]

ENTRYPOINT ["node", "dist/server.js"]
```

### Complete Secure Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: production
  labels:
    app: secure-app
    version: v1.0.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
      annotations:
        container.apparmor.security.beta.kubernetes.io/app: runtime/default
    spec:
      automountServiceAccountToken: false
      serviceAccountName: secure-app-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        runAsGroup: 1001
        fsGroup: 1001
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: app
          image: myregistry.io/secure-app:v1.0.0@sha256:abc123...
          ports:
            - containerPort: 3000
              protocol: TCP
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: db-password
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir:
            sizeLimit: 50Mi
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: secure-app
```

### Pod Security Standards

Kubernetes defines three security profiles:

| Level | Description | Use Case |
|-------|-------------|----------|
| **Privileged** | No restrictions | System workloads (kube-system) |
| **Baseline** | Minimal restrictions, prevents known escalations | General workloads |
| **Restricted** | Heavily restricted, best practices | Security-sensitive workloads |

```yaml
# Enforce Pod Security Standards via namespace labels
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

---

## 1.4 Kubernetes Security

### RBAC Advanced Scenarios

```yaml
# Scenario: CI/CD service account that can only deploy to specific namespace
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-deployer
  namespace: staging
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployer-role
  namespace: staging
rules:
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["services", "configmaps"]
    verbs: ["get", "list", "create", "update", "patch"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "list", "create", "update", "patch"]
  # Explicitly NO access to secrets, nodes, or other namespaces
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-deployer-binding
  namespace: staging
subjects:
  - kind: ServiceAccount
    name: cicd-deployer
    namespace: staging
roleRef:
  kind: Role
  name: deployer-role
  apiGroup: rbac.authorization.k8s.io
```

### Network Policies

```yaml
# Default deny all traffic in namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

---
# Allow frontend to talk to backend only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080

---
# Allow backend to access database only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-egress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: postgres
      ports:
        - protocol: TCP
          port: 5432
    - to:  # Allow DNS
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

### OPA/Gatekeeper Policy Enforcement

```yaml
# Constraint Template: Require resource limits
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlimits
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLimits
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlimits
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
          msg := sprintf("Container %v must have memory limits", [container.name])
        }
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.cpu
          msg := sprintf("Container %v must have CPU limits", [container.name])
        }

---
# Apply the constraint
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLimits
metadata:
  name: must-have-limits
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
    namespaces: ["production", "staging"]
```

### Falco Runtime Threat Detection

```yaml
# Falco rule examples
- rule: Terminal shell in container
  desc: Detect shell opened in a container
  condition: >
    spawned_process and container and
    shell_procs and proc.tty != 0
  output: >
    Shell opened in container
    (user=%user.name container=%container.name shell=%proc.name)
  priority: WARNING

- rule: Read sensitive file
  desc: Detect read of sensitive files
  condition: >
    open_read and container and
    (fd.name startswith /etc/shadow or fd.name startswith /etc/passwd)
  output: >
    Sensitive file read in container
    (file=%fd.name container=%container.name)
  priority: CRITICAL
```

### Secret Management in Kubernetes

**Problem: Native K8s secrets are NOT encrypted — they're base64 encoded:**
```bash
# Anyone with RBAC access can decode
echo "cGFzc3dvcmQxMjM=" | base64 -d  # password123
```

**Solution options:**

**1. External Secrets Operator:**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: app-secrets
    creationPolicy: Owner
  data:
    - secretKey: db-password
      remoteRef:
        key: production/database
        property: password
    - secretKey: api-key
      remoteRef:
        key: production/api-keys
        property: main-key
```

**2. Sealed Secrets (Bitnami):**
```bash
# Encrypt a secret (can be safely committed to Git)
kubectl create secret generic my-secret \
  --from-literal=password=supersecret \
  --dry-run=client -o yaml | \
  kubeseal --controller-name=sealed-secrets \
  --controller-namespace=kube-system \
  --format yaml > sealed-secret.yaml
```

**3. CSI Secret Store Driver:**
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kv-secrets
spec:
  provider: azure
  parameters:
    keyvaultName: "mycompany-kv"
    objects: |
      array:
        - |
          objectName: db-connection-string
          objectType: secret
    tenantId: "your-tenant-id"
```

### Service Mesh mTLS

```yaml
# Istio: Enforce strict mTLS for namespace
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # All traffic must be mTLS

---
# Istio: Authorization policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: frontend-to-backend
  namespace: production
spec:
  selector:
    matchLabels:
      app: backend
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/production/sa/frontend"]
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/*"]
```

---

## 1.5 Secret Management Deep Dive

### Why Secret Management Matters

- Secrets in code/config repos = #1 cause of breaches
- Static secrets that never rotate = ticking time bombs
- Shared secrets across environments = blast radius multiplication
- No audit trail = unable to investigate incidents

### HashiCorp Vault

**Architecture:**
```
┌─────────────────────────────────────────────────────┐
│                    Vault Server                       │
├─────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │  Auth    │  │  Secret  │  │  Audit           │  │
│  │  Methods │  │  Engines │  │  Devices         │  │
│  ├──────────┤  ├──────────┤  ├──────────────────┤  │
│  │ Token    │  │ KV v2    │  │ File             │  │
│  │ K8s SA   │  │ Database │  │ Syslog           │  │
│  │ AWS IAM  │  │ PKI      │  │ Socket           │  │
│  │ OIDC     │  │ Transit  │  │                  │  │
│  │ AppRole  │  │ AWS      │  │                  │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
├─────────────────────────────────────────────────────┤
│              Storage Backend (Consul/Raft)            │
└─────────────────────────────────────────────────────┘
```

**Dynamic Secrets (database example):**
```bash
# Enable database secret engine
vault secrets enable database

# Configure PostgreSQL connection
vault write database/config/mydb \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@db:5432/mydb" \
  allowed_roles="readonly,readwrite" \
  username="vault_admin" \
  password="vault_password"

# Create a role for dynamic credentials
vault write database/roles/readonly \
  db_name=mydb \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Get dynamic credentials (auto-expires!)
vault read database/creds/readonly
# Returns: username=v-token-readonly-abc123, password=random-generated
```

**Vault Policy:**
```hcl
# policy.hcl — Application can only read its own secrets
path "secret/data/production/myapp/*" {
  capabilities = ["read", "list"]
}

path "database/creds/myapp-readonly" {
  capabilities = ["read"]
}

# Deny access to other apps' secrets
path "secret/data/production/other-app/*" {
  capabilities = ["deny"]
}
```

**Vault + Kubernetes Integration:**
```yaml
# Vault Agent Injector annotations
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "myapp"
        vault.hashicorp.com/agent-inject-secret-db-creds: "database/creds/readonly"
        vault.hashicorp.com/agent-inject-template-db-creds: |
          {{- with secret "database/creds/readonly" -}}
          export DB_USER="{{ .Data.username }}"
          export DB_PASS="{{ .Data.password }}"
          {{- end -}}
    spec:
      serviceAccountName: myapp
      containers:
        - name: app
          image: myapp:v1.0
```

### AWS Secrets Manager

```bash
# Create a secret
aws secretsmanager create-secret \
  --name production/database/credentials \
  --secret-string '{"username":"admin","password":"s3cur3P@ss"}'

# Retrieve a secret
aws secretsmanager get-secret-value \
  --secret-id production/database/credentials

# Enable automatic rotation (Lambda-based)
aws secretsmanager rotate-secret \
  --secret-id production/database/credentials \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456:function:rotate-db \
  --rotation-rules AutomaticallyAfterDays=30
```

### Secret Rotation Strategies

| Strategy | Frequency | Complexity | Use Case |
|----------|-----------|-----------|----------|
| Manual rotation | Quarterly | Low | Low-risk, few secrets |
| Automated rotation | 30-90 days | Medium | Databases, API keys |
| Dynamic secrets | Per-request | High | High-security, Vault |
| Short-lived tokens | Minutes-hours | Medium | Service-to-service |


---

## 1.6 Network Security

### Firewall Rules Design

```
Principle: Default deny, explicit allow

┌─────────────────────────────────────────────────────────────┐
│ Priority │  Direction │  Source      │  Dest    │  Action   │
├──────────┼────────────┼──────────────┼──────────┼───────────┤
│ 100      │  Inbound   │  LB subnet   │  App:443 │  ALLOW    │
│ 200      │  Inbound   │  App subnet  │  DB:5432 │  ALLOW    │
│ 300      │  Inbound   │  VPN CIDR    │  SSH:22  │  ALLOW    │
│ 65535    │  Inbound   │  0.0.0.0/0   │  Any     │  DENY     │
│ 100      │  Outbound  │  App subnet  │  DB:5432 │  ALLOW    │
│ 200      │  Outbound  │  Any         │  443     │  ALLOW    │
│ 65535    │  Outbound  │  Any         │  Any     │  DENY     │
└──────────┴────────────┴──────────────┴──────────┴───────────┘
```

**Terraform example:**
```hcl
# AWS Security Group — least privilege
resource "aws_security_group" "app" {
  name_prefix = "app-"
  vpc_id      = var.vpc_id

  # Only allow traffic from ALB
  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  # Only allow outbound to database and HTTPS
  egress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.db.id]
  }

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS for external APIs"
  }
}
```

### Web Application Firewalls (WAF)

```hcl
# AWS WAF with common rules
resource "aws_wafv2_web_acl" "main" {
  name  = "app-waf"
  scope = "REGIONAL"

  default_action { allow {} }

  # Block SQL injection
  rule {
    name     = "sql-injection"
    priority = 1
    override_action { none {} }
    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesSQLiRuleSet"
      }
    }
    visibility_config {
      sampled_requests_enabled   = true
      cloudwatch_metrics_enabled = true
      metric_name                = "SQLiRule"
    }
  }

  # Block known bad inputs
  rule {
    name     = "known-bad-inputs"
    priority = 2
    override_action { none {} }
    statement {
      managed_rule_group_statement {
        vendor_name = "AWS"
        name        = "AWSManagedRulesKnownBadInputsRuleSet"
      }
    }
    visibility_config {
      sampled_requests_enabled   = true
      cloudwatch_metrics_enabled = true
      metric_name                = "BadInputsRule"
    }
  }

  # Rate limiting
  rule {
    name     = "rate-limit"
    priority = 3
    action { block {} }
    statement {
      rate_based_statement {
        limit              = 2000
        aggregate_key_type = "IP"
      }
    }
    visibility_config {
      sampled_requests_enabled   = true
      cloudwatch_metrics_enabled = true
      metric_name                = "RateLimitRule"
    }
  }
}
```

### TLS Everywhere with cert-manager

```yaml
# Install cert-manager and configure Let's Encrypt
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: devops@company.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
      - http01:
          ingress:
            class: nginx

---
# Automatic TLS for ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - app.company.com
      secretName: app-tls-cert
  rules:
    - host: app.company.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

---

## 1.7 OWASP Top 10 (DevOps Relevance)

### A01: Broken Access Control

**What:** Users acting outside their intended permissions.
**DevOps relevance:** Misconfigured RBAC, overly permissive IAM roles, exposed admin endpoints.
**Prevention:**
- Implement least privilege in all IAM/RBAC configs
- Default deny access controls
- Automated RBAC auditing in CI/CD
- Test access control in DAST scans

### A02: Cryptographic Failures

**What:** Failures related to cryptography leading to data exposure.
**DevOps relevance:** Secrets in environment variables/logs, weak TLS configs, unencrypted data at rest.
**Prevention:**
- Enforce TLS 1.2+ everywhere
- Use secret management tools (Vault, AWS Secrets Manager)
- Encrypt data at rest (database encryption, encrypted volumes)
- Scan for hardcoded secrets in CI/CD

### A03: Injection

**What:** Hostile data sent to an interpreter (SQL, OS, LDAP injection).
**DevOps relevance:** CI/CD scripts vulnerable to injection, unsanitized variables in pipelines.
**Prevention:**
- SAST tools to detect injection vulnerabilities
- Parameterized queries (enforce in code reviews)
- Input validation at all layers
- WAF rules for common injection patterns

### A04: Insecure Design

**What:** Missing or ineffective security controls in design.
**DevOps relevance:** Architecture without threat modeling, no defense in depth.
**Prevention:**
- Threat modeling in design phase
- Security architecture reviews
- Abuse case testing in CI/CD
- Design patterns: rate limiting, circuit breakers

### A05: Security Misconfiguration

**What:** Incorrectly configured security settings.
**DevOps relevance:** Default credentials, open cloud storage, verbose error messages, unnecessary features enabled.
**Prevention:**
- IaC scanning (Checkov, tfsec) in every pipeline
- CIS benchmarks (kube-bench)
- Automated configuration compliance
- Hardened base images and templates

### A06: Vulnerable and Outdated Components

**What:** Using components with known vulnerabilities.
**DevOps relevance:** Outdated base images, unpatched dependencies, EOL software.
**Prevention:**
- SCA scanning in CI/CD (Snyk, npm audit)
- Container image scanning (Trivy)
- Automated dependency updates (Dependabot, Renovate)
- SBOM tracking

### A07: Identification and Authentication Failures

**What:** Weaknesses in authentication mechanisms.
**DevOps relevance:** Weak CI/CD authentication, shared credentials, no MFA on admin access.
**Prevention:**
- MFA on all administrative access
- Short-lived tokens for CI/CD
- SSO integration for all tools
- Regular credential rotation

### A08: Software and Data Integrity Failures

**What:** Code and infrastructure that doesn't protect against integrity violations.
**DevOps relevance:** Unsigned images, untrusted CI/CD plugins, supply chain attacks.
**Prevention:**
- Image signing (Cosign)
- Verified base images
- Lock file integrity (package-lock.json)
- Pipeline integrity (signed commits, protected branches)

### A09: Security Logging and Monitoring Failures

**What:** Insufficient logging to detect breaches.
**DevOps relevance:** No audit logs, alerts not configured, logs not centralized.
**Prevention:**
- Centralized logging (ELK, Loki, CloudWatch)
- Security event alerting
- Audit logging for all admin actions
- SIEM integration

### A10: Server-Side Request Forgery (SSRF)

**What:** Application fetches remote resources without validating user-supplied URLs.
**DevOps relevance:** Cloud metadata endpoint access, internal service discovery abuse.
**Prevention:**
- Network segmentation
- Block metadata endpoints (169.254.169.254)
- Whitelist allowed outbound destinations
- DAST scanning for SSRF

---

## 1.8 GCC Compliance & Regulations

### SAMA (Saudi Arabian Monetary Authority) — Cyber Security Framework

**Key requirements for DevOps teams:**
- **Data classification** — All data must be classified and handled accordingly
- **Encryption** — Data at rest and in transit must be encrypted
- **Access control** — Strict role-based access with MFA
- **Logging** — All access and changes must be logged and retained (minimum 5 years)
- **Incident response** — Documented IR plan with regular drills
- **Third-party risk** — Cloud providers must meet SAMA requirements
- **Vulnerability management** — Regular scanning and patching (critical within 48 hours)

**DevOps implications:**
```yaml
# Example: SAMA-compliant logging configuration
# All deployments must have audit logging enabled
audit_logging:
  enabled: true
  retention_days: 1825  # 5 years
  encryption: AES-256
  tamper_protection: true
  fields:
    - timestamp
    - user_identity
    - action
    - resource
    - source_ip
    - result
```

### NCA/NESA (National Cybersecurity Authority — Saudi / UAE)

**Essential Cybersecurity Controls (ECC):**
- Asset management and inventory
- Identity and access management
- Data protection and privacy
- Network security management
- Application security (secure SDLC)
- Change management and DevOps security
- Incident and threat management
- Business continuity

### PDPL (Saudi Personal Data Protection Law)

**Key requirements:**
- Explicit consent for data collection
- Data minimization principle
- Right to access, correct, and delete personal data
- Data breach notification within 72 hours
- Data residency — personal data of Saudi citizens must remain in KSA (with exceptions)
- Privacy impact assessments for new systems

**DevOps implications:**
- Infrastructure must be deployed in-region (AWS Bahrain, Azure UAE/Qatar)
- Implement data classification labels in metadata
- Automated PII detection in CI/CD pipelines
- Data retention policies enforced programmatically

### Qatar National Information and Cyber Security (NICS)

- Similar to NCA framework
- Focus on critical infrastructure protection
- Mandatory security assessments for government systems
- Data sovereignty requirements

### Data Residency Requirements

| Country | Requirement | Cloud Options |
|---------|-------------|---------------|
| Saudi Arabia | KSA for government, financial | AWS (Bahrain), Azure (Jeddah planned), Oracle (Jeddah) |
| UAE | UAE for government data | Azure (UAE North/Central), AWS (UAE planned) |
| Qatar | Qatar for sensitive data | Azure (Qatar), AWS (Bahrain proximity) |
| Bahrain | Less strict | AWS (Bahrain) |

### What DevOps Teams Must Do for Compliance

1. **Infrastructure:** Deploy in approved regions only — enforce via IaC policies
2. **Encryption:** TLS 1.2+ in transit, AES-256 at rest — enforce via security groups
3. **Access:** MFA everywhere, RBAC with least privilege, regular access reviews
4. **Logging:** Centralized, immutable, retained per regulation
5. **Scanning:** Regular vulnerability scanning with SLA-based remediation
6. **DR/BCP:** Tested disaster recovery within approved regions
7. **Change Management:** Documented, approved changes (GitOps provides this naturally)

---

## 1.9 Incident Response

### Security Incident Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│PREPARATION│───▶│DETECTION │───▶│CONTAINMENT───▶│ERADICATION───▶│ RECOVERY │
│           │    │& ANALYSIS│    │           │    │           │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                                       │
                                                                       ▼
                                                              ┌──────────────┐
                                                              │LESSONS LEARNED│
                                                              └──────────────┘
```

**DevOps-specific incident response:**

| Phase | DevOps Actions |
|-------|---------------|
| Preparation | Runbooks, automated alerting, IR drills |
| Detection | SIEM alerts, anomaly detection, Falco alerts |
| Containment | Network policies to isolate, scale down compromised pods |
| Eradication | Rebuild from clean images, rotate all credentials |
| Recovery | Deploy from known-good state (GitOps revert), verify |
| Lessons | Blameless postmortem, update IaC/pipelines |

### SIEM Basics

**Azure Sentinel (Microsoft Sentinel):**
```kql
// KQL query: Detect unusual container image pulls
ContainerImagePull_CL
| where TimeGenerated > ago(1h)
| where Registry_s !in ("approved-registry.azurecr.io", "mcr.microsoft.com")
| project TimeGenerated, Registry_s, Image_s, Namespace_s
| summarize Count=count() by Registry_s, Image_s
```

**Splunk query example:**
```spl
index=kubernetes sourcetype=kube:audit
| where verb="create" AND objectRef.resource="secrets"
| stats count by user.username, objectRef.namespace
| where count > 10
| sort -count
```

### Forensics Basics for DevOps

```bash
# Container forensics — capture state before killing
kubectl exec compromised-pod -- ps aux > /evidence/processes.txt
kubectl exec compromised-pod -- netstat -tlnp > /evidence/network.txt
kubectl exec compromised-pod -- find / -newer /tmp/baseline -type f > /evidence/modified-files.txt

# Copy container filesystem for analysis
kubectl cp compromised-pod:/app /evidence/app-filesystem/

# Get pod events and logs
kubectl describe pod compromised-pod > /evidence/pod-describe.txt
kubectl logs compromised-pod --all-containers > /evidence/pod-logs.txt

# Snapshot the node for disk forensics (cloud)
aws ec2 create-snapshot --volume-id vol-xxx --description "Forensics - Incident #123"
```

### Communication During Security Incidents

**Internal communication:**
- Dedicated incident channel (Slack/Teams)
- Clear roles: Incident Commander, Technical Lead, Communications Lead
- Regular status updates (every 30-60 minutes during active incident)
- Document all actions taken

**External communication (if required by regulation):**
- SAMA: Notify within 72 hours for significant incidents
- PDPL: Notify within 72 hours for data breaches affecting personal data
- Prepare holding statement templates in advance

**Post-incident:**
- Blameless postmortem within 48 hours
- Root cause analysis
- Action items with owners and deadlines
- Share learnings across the organization


---

# SECTION 2: GITOPS WITH ARGOCD

---

## 2.1 GitOps Principles

### What is GitOps?

GitOps is an operational framework that uses Git as the single source of truth for declarative infrastructure and applications. The desired state of your system is stored in Git, and automated processes ensure the actual state matches.

**Core principles:**
1. **Declarative** — The entire system is described declaratively (YAML, HCL)
2. **Versioned and immutable** — The desired state is stored in Git (versioned, auditable)
3. **Pulled automatically** — Agents pull desired state and apply it
4. **Continuously reconciled** — Agents continuously observe and correct drift

### GitOps vs Traditional CI/CD

```
┌─── Traditional CI/CD (Push Model) ───────────────────────────┐
│                                                                │
│  Developer → Git Push → CI Build → CI Deploys to Cluster      │
│                                    (CI has cluster creds)      │
│                                                                │
│  Problems:                                                     │
│  - CI system has production credentials                        │
│  - No drift detection                                          │
│  - Rollback requires re-running pipeline                       │
│  - Cluster state may differ from Git                           │
└────────────────────────────────────────────────────────────────┘

┌─── GitOps (Pull Model) ──────────────────────────────────────┐
│                                                                │
│  Developer → Git Push → Config Repo updated                    │
│                              ↑                                 │
│                              │ (observes)                      │
│                              ↓                                 │
│  GitOps Agent (in cluster) → Pulls desired state → Applies    │
│                                                                │
│  Benefits:                                                     │
│  - No external credentials needed                              │
│  - Continuous drift detection and self-healing                 │
│  - Rollback = git revert                                       │
│  - Complete audit trail in Git history                          │
│  - Cluster state always matches Git                            │
└────────────────────────────────────────────────────────────────┘
```

### Benefits of GitOps

| Benefit | Description |
|---------|-------------|
| Audit trail | Every change is a Git commit with author, timestamp, message |
| Drift detection | Agent detects when cluster state differs from Git |
| Self-healing | Agent automatically corrects drift |
| Rollback | `git revert` = deployment rollback |
| Security | No external system needs cluster credentials |
| Consistency | All environments managed the same way |
| Developer experience | Familiar Git workflow for operations |

### GitOps Operators

| Feature | ArgoCD | Flux |
|---------|--------|------|
| UI | Rich web UI | No built-in UI |
| Multi-cluster | Native support | Via Kustomization |
| Helm support | Native | HelmRelease CRD |
| RBAC | Fine-grained | K8s native |
| SSO | Built-in | Via external tools |
| Image updates | Image Updater addon | Built-in |
| Community | Larger | Growing |
| CNCF status | Graduated | Graduated |

---

## 2.2 ArgoCD Deep Dive

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        ArgoCD Architecture                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐     ┌──────────────┐     ┌────────────────┐   │
│  │   API Server │     │  Repo Server │     │  Application   │   │
│  │              │     │              │     │  Controller    │   │
│  │ - REST/gRPC  │     │ - Git clone  │     │               │   │
│  │ - Web UI     │     │ - Manifest   │     │ - Reconcile    │   │
│  │ - Auth/RBAC  │     │   generation │     │ - Sync         │   │
│  │ - CLI API    │     │ - Helm/Kust  │     │ - Health check │   │
│  │              │     │   rendering  │     │ - Drift detect │   │
│  └──────┬───────┘     └──────┬───────┘     └───────┬────────┘   │
│         │                     │                      │            │
│         └─────────────────────┼──────────────────────┘            │
│                               │                                   │
│                    ┌──────────▼──────────┐                        │
│                    │   Redis (Cache)     │                        │
│                    └─────────────────────┘                        │
│                                                                   │
│  ┌────────────────┐              ┌────────────────────────────┐  │
│  │  Git Repos     │              │  Kubernetes Cluster(s)     │  │
│  │  (Source)      │              │  (Destination)             │  │
│  └────────────────┘              └────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Installation

```bash
# Method 1: Plain YAML (simple, good for learning)
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Method 2: Helm (recommended for production)
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --set server.service.type=LoadBalancer \
  --set configs.params."server\.insecure"=true \
  --values argocd-values.yaml

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Install ArgoCD CLI
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd && sudo mv argocd /usr/local/bin/

# Login
argocd login argocd.example.com --grpc-web
```

### Application CRD (Complete Example)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-application
  namespace: argocd
  labels:
    team: backend
    env: production
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: production

  source:
    repoURL: https://github.com/company/k8s-configs.git
    targetRevision: main
    path: apps/my-application/overlays/production

    # For Helm charts:
    # chart: my-chart
    # helm:
    #   valueFiles:
    #     - values-production.yaml
    #   parameters:
    #     - name: image.tag
    #       value: "v1.2.3"

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true        # Delete resources removed from Git
      selfHeal: true     # Fix drift automatically
      allowEmpty: false  # Don't sync if manifests are empty
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
      - ApplyOutOfSyncOnly=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas  # Ignore HPA-managed replicas

  info:
    - name: url
      value: https://my-app.company.com
```

### Sync Policies

```yaml
# Manual sync — requires human approval
syncPolicy: {}

# Automated sync — auto-applies changes from Git
syncPolicy:
  automated:
    prune: true      # Remove resources deleted from Git
    selfHeal: true   # Revert manual changes to cluster

# Sync waves — control deployment order
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"   # Deploy first (namespaces, CRDs)
---
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"   # Deploy second (configs, secrets)
---
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "2"   # Deploy third (applications)
```

### Projects (Access Control)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications

  # Only these repos can be used
  sourceRepos:
    - 'https://github.com/company/k8s-configs.git'
    - 'https://github.com/company/helm-charts.git'

  # Only deploy to these clusters/namespaces
  destinations:
    - namespace: 'production'
      server: 'https://kubernetes.default.svc'
    - namespace: 'production-*'
      server: 'https://kubernetes.default.svc'

  # Deny certain resource types
  clusterResourceBlacklist:
    - group: ''
      kind: Namespace
    - group: rbac.authorization.k8s.io
      kind: ClusterRole

  # Allow only specific resource types
  namespaceResourceWhitelist:
    - group: 'apps'
      kind: Deployment
    - group: ''
      kind: Service
    - group: 'networking.k8s.io'
      kind: Ingress

  # Who can manage apps in this project
  roles:
    - name: developer
      description: Developer access
      policies:
        - p, proj:production:developer, applications, get, production/*, allow
        - p, proj:production:developer, applications, sync, production/*, allow
      groups:
        - dev-team

    - name: admin
      description: Full admin access
      policies:
        - p, proj:production:admin, applications, *, production/*, allow
      groups:
        - platform-team
```

### RBAC in ArgoCD

```yaml
# argocd-rbac-cm ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    # Roles
    p, role:dev-team, applications, get, */*, allow
    p, role:dev-team, applications, sync, */*, allow
    p, role:dev-team, logs, get, */*, allow

    p, role:platform-admin, applications, *, */*, allow
    p, role:platform-admin, clusters, *, *, allow
    p, role:platform-admin, repositories, *, *, allow
    p, role:platform-admin, projects, *, *, allow

    # Group bindings (from SSO)
    g, dev-team-group, role:dev-team
    g, platform-group, role:platform-admin
```

---

## 2.3 GitOps Repository Structure

### Mono-repo vs Multi-repo

```
# Mono-repo (all environments in one repo)
k8s-configs/
├── apps/
│   ├── frontend/
│   │   ├── base/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── staging/
│   │       └── production/
│   └── backend/
│       ├── base/
│       └── overlays/
├── infrastructure/
│   ├── cert-manager/
│   ├── ingress-nginx/
│   └── monitoring/
└── argocd/
    ├── projects/
    └── applications/

# Multi-repo (separate repos per concern)
# Repo 1: app-source (application code + Dockerfile)
# Repo 2: k8s-configs (Kubernetes manifests)
# Repo 3: infra-configs (infrastructure components)
```

### Using Kustomize with ArgoCD

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - hpa.yaml

# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
namespace: production
namePrefix: prod-
patches:
  - target:
      kind: Deployment
      name: app
    patch: |
      - op: replace
        path: /spec/replicas
        value: 3
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: myregistry.io/app:v1.2.3
  - target:
      kind: Deployment
      name: app
    patch: |
      - op: replace
        path: /spec/template/spec/containers/0/resources/limits/memory
        value: "512Mi"
```

### ApplicationSets

```yaml
# Deploy to multiple clusters using generators
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-app-set
  namespace: argocd
spec:
  generators:
    # List generator — explicit environments
    - list:
        elements:
          - cluster: dev
            url: https://dev-cluster.example.com
            namespace: app-dev
          - cluster: staging
            url: https://staging-cluster.example.com
            namespace: app-staging
          - cluster: production
            url: https://prod-cluster.example.com
            namespace: app-prod

  template:
    metadata:
      name: 'myapp-{{cluster}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/company/k8s-configs.git
        targetRevision: main
        path: 'apps/myapp/overlays/{{cluster}}'
      destination:
        server: '{{url}}'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true

---
# Git generator — auto-discover apps from directory structure
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-addons
  namespace: argocd
spec:
  generators:
    - git:
        repoURL: https://github.com/company/k8s-configs.git
        revision: main
        directories:
          - path: infrastructure/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: infrastructure
      source:
        repoURL: https://github.com/company/k8s-configs.git
        targetRevision: main
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
```

---

## 2.4 GitOps Workflows

### Image Update Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitOps Image Update Flow                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Developer pushes code to app repo                            │
│  2. CI pipeline builds and pushes image (tag: v1.2.3)            │
│  3. CI pipeline updates image tag in config repo                 │
│  4. ArgoCD detects change in config repo                         │
│  5. ArgoCD syncs new image to cluster                            │
│                                                                   │
│  App Repo          Config Repo              Cluster              │
│  ────────          ───────────              ───────              │
│  [code push] ───▶ [CI builds] ───▶ [update tag] ───▶ [sync]    │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

**CI pipeline step to update config repo:**
```yaml
# GitLab CI — update image tag in config repo
update-manifests:
  stage: deploy
  image: alpine/git
  script:
    - git clone https://oauth2:${GIT_TOKEN}@github.com/company/k8s-configs.git
    - cd k8s-configs
    - |
      sed -i "s|image: myregistry.io/myapp:.*|image: myregistry.io/myapp:${CI_COMMIT_SHA}|" \
        apps/myapp/overlays/staging/kustomization.yaml
    - git config user.email "ci@company.com"
    - git config user.name "CI Pipeline"
    - git add .
    - git commit -m "chore: update myapp image to ${CI_COMMIT_SHA}"
    - git push
  only:
    - main
```

### ArgoCD Image Updater

```yaml
# Automatically update image tags without CI modifying config repo
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  annotations:
    argocd-image-updater.argoproj.io/image-list: app=myregistry.io/myapp
    argocd-image-updater.argoproj.io/app.update-strategy: semver
    argocd-image-updater.argoproj.io/app.semver-constraint: ">=1.0.0 <2.0.0"
    argocd-image-updater.argoproj.io/write-back-method: git
    argocd-image-updater.argoproj.io/write-back-target: kustomization
```

### PR-Based Promotion

```yaml
# GitHub Actions: Create PR for production promotion
name: Promote to Production
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to promote'
        required: true

jobs:
  promote:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          repository: company/k8s-configs
          token: ${{ secrets.GIT_TOKEN }}

      - name: Update production overlay
        run: |
          cd apps/myapp/overlays/production
          kustomize edit set image myregistry.io/myapp:${{ inputs.version }}

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          title: "Promote myapp ${{ inputs.version }} to production"
          body: |
            ## Production Deployment
            - **Version:** ${{ inputs.version }}
            - **Tested in:** staging
            - **Approved by:** ${{ github.actor }}
          branch: promote/myapp-${{ inputs.version }}
          labels: production,deployment
```

### Rollback

```bash
# GitOps rollback = git revert
git revert HEAD  # Revert the last commit (the bad deployment)
git push         # ArgoCD automatically syncs the previous version

# Or use ArgoCD CLI
argocd app rollback my-app  # Roll back to previous successful sync

# View history
argocd app history my-app
```

---

## 2.5 Secret Management in GitOps

### Problem: Secrets Can't Be in Git

Git is designed to be accessible and shareable. Secrets in Git = secrets everywhere the repo is cloned.

### Sealed Secrets (Bitnami)

```bash
# Install controller
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets -n kube-system

# Install kubeseal CLI
brew install kubeseal  # or download from GitHub

# Create and seal a secret
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password=s3cretP@ss \
  --dry-run=client -o yaml | \
  kubeseal --format yaml > sealed-db-creds.yaml

# The sealed secret is safe to commit to Git!
cat sealed-db-creds.yaml
```

```yaml
# sealed-db-creds.yaml (safe to commit)
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: db-creds
  namespace: production
spec:
  encryptedData:
    username: AgBy3i4OJSWK+PiTySYZZ... # encrypted
    password: AgCtr8fSksDF+JKwlzkQ3... # encrypted
```

### External Secrets Operator

```yaml
# SecretStore — defines where secrets live
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: me-south-1  # Bahrain (GCC region)
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
            namespace: external-secrets

---
# ExternalSecret — what to fetch
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: app-secrets
  data:
    - secretKey: database-url
      remoteRef:
        key: production/myapp/database
        property: connection_string
    - secretKey: api-key
      remoteRef:
        key: production/myapp/api-keys
        property: main
```

### SOPS (Mozilla)

```bash
# Encrypt a secret file with SOPS
sops --encrypt --age $(cat keys.txt) secrets.yaml > secrets.enc.yaml

# Decrypt
sops --decrypt secrets.enc.yaml > secrets.yaml

# Use with ArgoCD + Helm Secrets plugin
# The SOPS-encrypted file can be committed to Git
```

```yaml
# secrets.enc.yaml (safe to commit — encrypted)
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
data:
  password: ENC[AES256_GCM,data:encrypted_base64_here,type:str]
sops:
  kms:
    - arn: arn:aws:kms:me-south-1:123456:key/abc-123
  encrypted_regex: ^(data|stringData)$
```

---

## 2.6 Complete GitOps Setup

### Full Example: ArgoCD + Kustomize + Multi-Environment

**Repository structure:**
```
k8s-configs/
├── apps/
│   └── webapp/
│       ├── base/
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   ├── ingress.yaml
│       │   └── kustomization.yaml
│       └── overlays/
│           ├── dev/
│           │   ├── kustomization.yaml
│           │   └── replicas-patch.yaml
│           ├── staging/
│           │   ├── kustomization.yaml
│           │   └── replicas-patch.yaml
│           └── production/
│               ├── kustomization.yaml
│               ├── replicas-patch.yaml
│               └── hpa.yaml
├── argocd/
│   ├── apps/
│   │   ├── webapp-dev.yaml
│   │   ├── webapp-staging.yaml
│   │   └── webapp-production.yaml
│   └── projects/
│       ├── dev-project.yaml
│       └── production-project.yaml
└── infrastructure/
    ├── cert-manager/
    ├── ingress-nginx/
    └── sealed-secrets/
```

**ArgoCD Application for each environment:**
```yaml
# argocd/apps/webapp-production.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: webapp-production
  namespace: argocd
spec:
  project: production
  source:
    repoURL: https://github.com/company/k8s-configs.git
    targetRevision: main
    path: apps/webapp/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
      - CreateNamespace=true
```

**CI Pipeline updates config repo (GitLab CI):**
```yaml
stages:
  - build
  - test
  - security
  - publish
  - update-manifests

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

security-scan:
  stage: security
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

update-staging:
  stage: update-manifests
  script:
    - git clone https://oauth2:${CONFIG_REPO_TOKEN}@github.com/company/k8s-configs.git
    - cd k8s-configs/apps/webapp/overlays/staging
    - kustomize edit set image myregistry.io/webapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - git add . && git commit -m "deploy: webapp $CI_COMMIT_SHA to staging"
    - git push
  only:
    - main
```

### Monitoring ArgoCD

```yaml
# ServiceMonitor for Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
  endpoints:
    - port: metrics
---
# Key ArgoCD metrics to alert on:
# argocd_app_sync_status{sync_status="OutOfSync"} — apps not matching Git
# argocd_app_health_status{health_status="Degraded"} — unhealthy apps
# argocd_git_request_duration_seconds — Git fetch latency
```


---

# SECTION 3: CAPSTONE PROJECT

---

## 3.1 Project Requirements

Build a production-ready, end-to-end DevOps platform demonstrating:

| Component | Requirement |
|-----------|-------------|
| Infrastructure | Terraform-managed cloud resources (VPC, EKS/AKS, RDS, etc.) |
| Application | Containerized multi-tier app (frontend + backend + database) |
| CI/CD | Automated pipeline (GitLab CI or Azure DevOps) |
| GitOps | ArgoCD managing deployments |
| Monitoring | Prometheus + Grafana + alerting |
| Security | SAST, container scanning, RBAC, Network Policies, secrets management |
| Documentation | Architecture docs, runbooks, README |

---

## 3.2 Project Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        CAPSTONE ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────┐     ┌──────────────┐     ┌────────────────────────────┐  │
│  │Developer │────▶│  GitLab/ADO  │────▶│  Container Registry        │  │
│  │          │     │  CI Pipeline │     │  (ECR/ACR/GitLab Registry) │  │
│  └──────────┘     └──────┬───────┘     └────────────┬───────────────┘  │
│                           │                           │                   │
│                           ▼                           │                   │
│                   ┌───────────────┐                   │                   │
│                   │  Config Repo  │                   │                   │
│                   │  (K8s YAML)   │                   │                   │
│                   └───────┬───────┘                   │                   │
│                           │                           │                   │
│                           ▼                           ▼                   │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    Kubernetes Cluster (EKS/AKS)                   │   │
│  │                                                                   │   │
│  │  ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌───────────────────┐  │   │
│  │  │ ArgoCD  │  │Frontend │  │ Backend  │  │  PostgreSQL (RDS) │  │   │
│  │  │         │  │ (React) │  │ (Node.js)│  │                   │  │   │
│  │  └─────────┘  └─────────┘  └──────────┘  └───────────────────┘  │   │
│  │                                                                   │   │
│  │  ┌─────────────────┐  ┌──────────────┐  ┌────────────────────┐  │   │
│  │  │  Prometheus +   │  │   Ingress    │  │  Sealed Secrets /  │  │   │
│  │  │  Grafana        │  │   (NGINX)    │  │  External Secrets  │  │   │
│  │  └─────────────────┘  └──────────────┘  └────────────────────┘  │   │
│  │                                                                   │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    Terraform-Managed Infrastructure                │   │
│  │  VPC │ Subnets │ Security Groups │ EKS/AKS │ RDS │ S3/Blob      │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3.3 Step-by-Step Implementation Guide

### Phase 1: Infrastructure (Terraform)

```hcl
# main.tf — EKS cluster with networking
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"

  name = "capstone-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["me-south-1a", "me-south-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true  # Cost optimization for non-prod
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.16.0"

  cluster_name    = "capstone-cluster"
  cluster_version = "1.28"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true

  eks_managed_node_groups = {
    general = {
      instance_types = ["t3.medium"]
      min_size       = 2
      max_size       = 4
      desired_size   = 2
    }
  }
}
```

### Phase 2: Application Containerization

```dockerfile
# backend/Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM gcr.io/distroless/nodejs18-debian12:nonroot
COPY --from=builder --chown=nonroot:nonroot /app/dist /app
COPY --from=builder --chown=nonroot:nonroot /app/node_modules /app/node_modules
WORKDIR /app
USER nonroot
EXPOSE 3000
CMD ["server.js"]
```

### Phase 3: CI/CD Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - test
  - security
  - build
  - update-manifests

unit-tests:
  stage: test
  script:
    - npm ci && npm test

sast-scan:
  stage: security
  script:
    - semgrep scan --config=auto --json -o sast.json .

container-build:
  stage: build
  script:
    - docker build -t $REGISTRY/backend:$CI_COMMIT_SHA .
    - trivy image --exit-code 1 --severity CRITICAL $REGISTRY/backend:$CI_COMMIT_SHA
    - docker push $REGISTRY/backend:$CI_COMMIT_SHA
    - cosign sign --key env://COSIGN_KEY $REGISTRY/backend:$CI_COMMIT_SHA

update-config:
  stage: update-manifests
  script:
    - git clone https://oauth2:$TOKEN@github.com/company/k8s-configs.git
    - cd k8s-configs && ./scripts/update-image.sh backend $CI_COMMIT_SHA
    - git commit -am "deploy: backend $CI_COMMIT_SHA" && git push
```

### Phase 4: GitOps Deployment

- Install ArgoCD in cluster
- Configure Application CRDs for each environment
- Set up automated sync with self-healing
- Configure notifications (Slack/Teams)

### Phase 5: Monitoring

- Deploy kube-prometheus-stack via Helm
- Create Grafana dashboards (application metrics, cluster health)
- Configure alerting rules (high error rate, pod crashes, resource exhaustion)
- Set up log aggregation (Loki or ELK)

### Phase 6: Security Hardening

- Implement Network Policies (default deny + allow specific)
- Configure Pod Security Standards (restricted)
- Set up External Secrets Operator
- Enable image scanning in pipeline (fail on CRITICAL)
- Implement RBAC for team access
- Configure OPA/Gatekeeper policies

### Phase 7: Documentation

- Architecture Decision Records (ADRs)
- Runbooks for common operations
- Disaster recovery plan
- Security documentation
- README with setup instructions

---

## 3.4 Evaluation Criteria

| Criteria | Weight | Description |
|----------|--------|-------------|
| Infrastructure as Code | 20% | Terraform quality, modularity, state management |
| CI/CD Pipeline | 20% | Completeness, security gates, efficiency |
| GitOps Implementation | 15% | ArgoCD config, multi-env, sync policies |
| Security | 20% | Scanning, RBAC, secrets, Network Policies |
| Monitoring | 15% | Metrics, dashboards, alerting |
| Documentation | 10% | Clarity, completeness, runbooks |

---

## 3.5 Portfolio Presentation Tips

1. **Create a GitHub README** with architecture diagram, screenshots, and tech stack badges
2. **Record a demo video** (5-10 minutes) walking through the pipeline end-to-end
3. **Write a blog post** explaining key decisions and trade-offs
4. **Highlight GCC relevance** — mention data residency, compliance considerations
5. **Show metrics** — deployment frequency, lead time, MTTR
6. **Include a "lessons learned" section** — shows maturity and self-awareness
7. **Make it reproducible** — include setup scripts and clear instructions

---

# SECTION 4: CAREER & INTERVIEW PREPARATION

---

## 4.1 Resume Building for DevOps

### What to Include

**Technical skills section (prioritize for GCC market):**
- Cloud: AWS, Azure (both are dominant in GCC)
- IaC: Terraform, ARM templates
- Containers: Docker, Kubernetes (EKS/AKS)
- CI/CD: Azure DevOps, GitLab CI, Jenkins
- Monitoring: Prometheus, Grafana, ELK, Azure Monitor
- Security: DevSecOps, Vault, container scanning
- GitOps: ArgoCD, Flux
- Scripting: Bash, Python, PowerShell

### Keywords for GCC Market

Include these terms naturally in your resume:
- Cloud migration, hybrid cloud
- Kubernetes, containerization, microservices
- DevSecOps, shift-left security
- Terraform, Infrastructure as Code
- CI/CD automation, GitOps
- High availability, disaster recovery
- Compliance (SAMA, NCA, PDPL)
- Data sovereignty, data residency
- Arabic language (if applicable — significant advantage)

### Project Descriptions (STAR Format)

```
BAD:  "Managed Kubernetes clusters"
GOOD: "Designed and deployed multi-AZ EKS cluster serving 50+ microservices,
       reducing deployment time from 2 hours to 15 minutes through GitOps
       (ArgoCD) implementation while maintaining SAMA compliance requirements"
```

**Quantify impact:**
- Reduced deployment time by X%
- Improved uptime from X to Y (99.9% → 99.99%)
- Reduced infrastructure costs by X%
- Managed X nodes / Y pods / Z deployments per day
- Decreased MTTR from X hours to Y minutes

---

## 4.2 Interview Preparation

### Types of DevOps Interviews

| Type | Duration | Focus |
|------|----------|-------|
| Technical screening | 30-45 min | Tool knowledge, scripting, troubleshooting |
| System design | 45-60 min | Architecture, scalability, trade-offs |
| Hands-on/live coding | 60-90 min | Build pipeline, write Terraform, debug |
| Behavioral | 30-45 min | Teamwork, conflict, leadership, communication |
| Culture fit | 30 min | Values alignment, work style |

### System Design Interview Approach

**Framework (use for any system design question):**

1. **Clarify requirements** (2-3 minutes)
   - Functional vs non-functional requirements
   - Scale: users, requests/second, data volume
   - Constraints: budget, compliance, region

2. **High-level design** (5-10 minutes)
   - Draw the architecture diagram
   - Identify main components
   - Show data flow

3. **Deep dive** (15-20 minutes)
   - Detail specific components
   - Discuss technology choices with trade-offs
   - Address scalability and reliability

4. **Security & Operations** (5-10 minutes)
   - How to deploy and update
   - Monitoring and alerting
   - Security controls
   - Disaster recovery

**Example question:** "Design a CI/CD platform for a company with 50 microservices"

### STAR Method for Behavioral Questions

| Letter | Meaning | Example |
|--------|---------|---------|
| **S** | Situation | "Our production system was experiencing frequent outages..." |
| **T** | Task | "I was tasked with improving reliability and reducing MTTR..." |
| **A** | Action | "I implemented comprehensive monitoring with Prometheus, created runbooks, set up PagerDuty rotation..." |
| **R** | Result | "Reduced MTTR from 4 hours to 20 minutes, improved uptime to 99.95%" |

### Common DevOps Scenarios

1. **"Your production deployment failed at 2 AM. Walk me through your response."**
2. **"How would you handle a security vulnerability in a running container?"**
3. **"A team wants to deploy directly to production without going through staging."**
4. **"The CI/CD pipeline takes 45 minutes. How do you optimize it?"**
5. **"You inherited a legacy monolith. What's your modernization strategy?"**

---

## 4.3 Salary Negotiation in GCC

### Market Ranges (2024-2025 estimates)

| Level | Saudi Arabia (SAR/month) | UAE (AED/month) | Qatar (QAR/month) |
|-------|-------------------------|-----------------|-------------------|
| Junior DevOps | 12,000-18,000 | 12,000-18,000 | 12,000-20,000 |
| Mid DevOps | 18,000-30,000 | 18,000-30,000 | 20,000-35,000 |
| Senior DevOps | 30,000-50,000 | 30,000-50,000 | 35,000-55,000 |
| Lead/Principal | 45,000-70,000+ | 45,000-70,000+ | 50,000-75,000+ |

**Negotiation tips:**
- Research market rates on Bayt.com, LinkedIn, GulfTalent
- Factor in: housing allowance, annual flight tickets, schooling allowance, end-of-service
- Total package > base salary (benefits can be 30-50% of base in GCC)
- Certifications (CKA, AWS SA, Azure) can increase offers 15-25%
- Highlight compliance knowledge (SAMA, NCA) — very valuable in GCC

---

## 4.4 Continuous Learning Strategy

### Quarterly Learning Plan

| Quarter | Focus | Certification Target |
|---------|-------|---------------------|
| Q1 | Cloud fundamentals + Linux | AWS SAA or AZ-104 |
| Q2 | Containers + Kubernetes | CKA |
| Q3 | CI/CD + IaC | Terraform Associate |
| Q4 | Security + Advanced topics | CKS or AWS Security Specialty |

### Learning Resources

- **Hands-on labs:** KodeKloud, A Cloud Guru, Katacoda alternatives
- **Practice:** Build personal projects, contribute to open source
- **Community:** Join local DevOps meetups, CNCF community groups
- **Stay current:** Follow CNCF landscape, DevOps weekly newsletters
- **Teach others:** Blog, present at meetups, mentor juniors


---

# SECTION 5: BEST PRACTICES

---

1. **Shift-left everything** — Security, quality, and compliance checks should happen as early as possible in the pipeline
2. **Automate security gates** — Never rely on manual security reviews for every deployment; automate with SAST/DAST/SCA
3. **Use GitOps for all environments** — Dev, staging, and production should all be managed through Git
4. **Separate app repos from config repos** — Application source code and Kubernetes manifests belong in different repositories
5. **Pin all versions** — Base images, dependencies, Helm chart versions, Terraform providers — pin everything
6. **Image signing is non-negotiable** — Sign all production images with Cosign and verify before deployment
7. **Default deny networking** — Start with deny-all Network Policies and explicitly allow required traffic
8. **Run containers as non-root** — No exceptions for production workloads
9. **Use distroless or minimal base images** — Smaller attack surface, fewer CVEs to manage
10. **Implement Pod Security Standards** — Use "restricted" profile for production namespaces
11. **Never store secrets in Git** — Use Sealed Secrets, External Secrets Operator, or SOPS
12. **Rotate secrets automatically** — Use Vault dynamic secrets or cloud-native rotation
13. **Monitor ArgoCD sync status** — Alert on OutOfSync and Degraded applications
14. **Use ApplicationSets for multi-cluster** — Don't manually create Application CRDs for each environment
15. **Implement RBAC at every layer** — Kubernetes, ArgoCD, CI/CD, cloud provider, Git repos
16. **Break the pipeline on critical CVEs** — Don't just report; fail the build
17. **Generate and store SBOMs** — Required for compliance and incident response
18. **Use sync waves in ArgoCD** — Ensure dependencies deploy before dependents
19. **Implement resource quotas and limits** — Prevent noisy neighbor and resource exhaustion
20. **Document everything as code** — ADRs, runbooks, and procedures in Git alongside infrastructure
21. **Test disaster recovery regularly** — A DR plan that hasn't been tested is just a wish
22. **Use multi-AZ deployments** — Single AZ = single point of failure
23. **Implement circuit breakers** — Prevent cascade failures between services
24. **Use admission webhooks** — Enforce policies at the API server level, not just in CI/CD

---

# SECTION 6: LESSONS LEARNED / PRO TIPS

---

1. **"It works on my machine" dies with containers** — But "it works in dev" can still haunt you. Ensure environment parity through Kustomize overlays, not manual tweaks.

2. **ArgoCD self-heal can fight with HPA** — Add `ignoreDifferences` for `/spec/replicas` when using HPA, or ArgoCD will keep resetting replica count.

3. **Sealed Secrets are cluster-scoped** — If you rebuild your cluster, you need the sealing key backup. Store it securely in Vault or cloud KMS.

4. **GitOps doesn't mean no CI** — You still need CI for building, testing, and scanning. GitOps replaces the CD "push" with a "pull" model.

5. **Start with baseline Pod Security, not restricted** — Going straight to restricted will break many Helm charts. Migrate gradually.

6. **Trivy finds CVEs you can't fix** — Some CVEs are in base image OS packages with no fix available. Use `.trivyignore` with justification, not blind suppression.

7. **Network Policies need a CNI that supports them** — Default kubenet doesn't enforce Network Policies. Use Calico, Cilium, or Azure CNI.

8. **Vault is powerful but complex** — For small teams, start with cloud-native secret managers (AWS Secrets Manager, Azure Key Vault) + External Secrets Operator.

9. **ArgoCD Application of Applications pattern** — Use a "root" Application that manages other Applications. Makes bootstrapping new clusters trivial.

10. **DAST in CI/CD needs a running environment** — You can't DAST test without deploying first. Use it in staging, not in build stage.

11. **Compliance is a feature, not a blocker** — In GCC, compliance knowledge (SAMA, NCA) is a career differentiator. Embrace it.

12. **Git commit messages are your deployment log** — Use conventional commits (`feat:`, `fix:`, `deploy:`) for clear audit trails.

13. **Multi-tenancy in K8s is harder than you think** — Namespace isolation alone isn't enough. Add Network Policies, Resource Quotas, and consider dedicated node pools.

14. **Cost optimization matters in GCC** — Cloud spending is scrutinized. Use spot/preemptible nodes for non-prod, right-size resources, implement cluster autoscaler.

15. **Arabic documentation wins points** — If you can document runbooks in Arabic for GCC clients, you're immediately more valuable than competitors.

16. **Don't over-engineer your first GitOps setup** — Start with a simple mono-repo, single cluster. Add complexity only when needed.

17. **Backup etcd and test restores** — Managed K8s handles this, but know the process for self-managed clusters.

---

# SECTION 7: INTERVIEW QUESTIONS

---

## Security Questions (10)

**Q1: What is the difference between SAST and DAST? When would you use each?**

**A:** SAST (Static Application Security Testing) analyzes source code without execution, catching vulnerabilities like SQL injection patterns, hardcoded secrets, and insecure functions early in development. DAST (Dynamic Application Security Testing) tests a running application by simulating attacks, finding runtime vulnerabilities like XSS, authentication flaws, and misconfigurations. Use SAST in the build stage (shift-left) and DAST in staging/pre-production (requires running application). They are complementary — SAST catches code-level issues, DAST catches deployment/runtime issues.

**Q2: How do you handle secrets in a Kubernetes environment?**

**A:** Never use plain Kubernetes secrets alone (they're only base64 encoded, not encrypted). Solutions include:
- External Secrets Operator to sync from AWS Secrets Manager/Azure Key Vault
- Sealed Secrets (Bitnami) for GitOps workflows — encrypts secrets that can be stored in Git
- HashiCorp Vault with agent injector for dynamic secrets
- CSI Secret Store Driver for mounting secrets as volumes
- Enable etcd encryption at rest for the cluster itself

**Q3: Explain the principle of least privilege with a CI/CD example.**

**A:** The CI/CD service account should only have permissions it absolutely needs. For example, a deployment pipeline should have: push access to container registry, read access to config repo, write access to specific namespace (not cluster-admin), and no access to production secrets during build. Implement with: scoped IAM roles, Kubernetes RBAC with namespace-specific roles, short-lived tokens (not permanent credentials), and separate service accounts per pipeline stage.

**Q4: What is a Software Bill of Materials (SBOM) and why does it matter?**

**A:** An SBOM is a comprehensive inventory of all components, libraries, and dependencies in your software — like a nutrition label. It matters because: enables rapid response to vulnerabilities (e.g., quickly identifying all systems affected by Log4Shell), required by regulations (US EO 14028, growing GCC requirements), enables supply chain security, and supports license compliance. Generate with tools like Syft, attach to images with Cosign, and scan SBOMs with Grype.

**Q5: How would you implement Zero Trust in a Kubernetes cluster?**

**A:** Implement in layers: Network — default deny Network Policies, mTLS between services (Istio/Linkerd), no pod-to-pod trust. Identity — Service accounts with minimal RBAC, short-lived tokens, OIDC authentication. Workload — Pod Security Standards (restricted), read-only filesystem, non-root containers, drop all capabilities. Data — Encrypt secrets at rest and in transit, external secret management. Admission — OPA/Gatekeeper policies to enforce standards, image signature verification.

**Q6: A critical CVE is found in your base image. Walk through your response.**

**A:** 1) Assess: Use container scanning (Trivy) to identify all affected images across environments. 2) Prioritize: Check CVSS score, exploitability, and whether the vulnerable component is accessible. 3) Remediate: Update base image to patched version, rebuild all affected images. 4) Validate: Run security scan on new images, verify the CVE is resolved. 5) Deploy: Push through pipeline (security gates should pass now). 6) Verify: Confirm updated images are running in all environments. 7) Document: Record in incident log, update image pinning strategy if needed.

**Q7: What are Pod Security Standards and how do you enforce them?**

**A:** Pod Security Standards define three profiles: Privileged (unrestricted), Baseline (prevents known escalations), and Restricted (heavily restricted, best practices). Enforce via Pod Security Admission (built into K8s 1.25+) by labeling namespaces with enforce/audit/warn modes. The restricted profile requires: runAsNonRoot, drop ALL capabilities, read-only root filesystem, no privilege escalation, seccomp RuntimeDefault profile. Start with audit mode to identify violations before enforcing.

**Q8: How do you secure a CI/CD pipeline itself?**

**A:** Protect the pipeline from: Supply chain attacks — verify dependencies (lock files, checksums), use trusted base images. Credential theft — use short-lived tokens, limit secret exposure to specific stages. Code injection — protect branch rules, require approvals for pipeline changes. Pipeline tampering — sign pipeline configurations, audit changes. Secrets exposure — never log secrets, use masked variables, scan pipeline output. Dependency confusion — use private registries, namespace packages properly.

**Q9: Explain defense in depth for a web application deployed on Kubernetes.**

**A:** Layer 1: Edge — WAF (block OWASP Top 10), DDoS protection, TLS termination. Layer 2: Network — Security Groups, Network Policies (deny all default), mTLS. Layer 3: Platform — Pod Security Standards, RBAC, admission webhooks. Layer 4: Application — Input validation, authentication/authorization, secure coding. Layer 5: Data — Encryption at rest, secrets management, data classification. Layer 6: Monitoring — SIEM, Falco runtime detection, audit logging. Each layer can stop an attack independently.

**Q10: What compliance considerations are specific to the GCC market?**

**A:** Key frameworks: SAMA (Saudi banking) — encryption, logging retention (5 years), vulnerability management SLAs. NCA/ECC (Saudi) — comprehensive cybersecurity controls for critical infrastructure. PDPL (Saudi data protection) — consent, data minimization, breach notification within 72 hours. Data residency — Saudi data must remain in KSA (deploy in AWS Bahrain/local regions). For DevOps: enforce region constraints in Terraform, implement comprehensive audit logging, automate compliance checks in CI/CD, maintain encryption everywhere, document all processes.

---

## GitOps/ArgoCD Questions (8)

**Q1: What is GitOps and how does it differ from traditional CI/CD?**

**A:** GitOps uses Git as the single source of truth for infrastructure and application state. The key difference is the deployment model: Traditional CI/CD uses a "push" model where the CI system pushes changes to the cluster (CI has cluster credentials). GitOps uses a "pull" model where an in-cluster agent (ArgoCD/Flux) pulls desired state from Git and applies it. Benefits: better security (no external cluster access needed), built-in drift detection, self-healing, audit trail (Git history), and rollback via git revert.

**Q2: Explain ArgoCD's architecture and the role of each component.**

**A:** Three main components: 1) API Server — handles UI, CLI, and API requests, manages authentication/RBAC, serves the web interface. 2) Repository Server — clones Git repos, generates Kubernetes manifests (renders Helm templates, applies Kustomize overlays), caches results. 3) Application Controller — monitors running applications, compares actual state vs desired state (from repo server), executes sync operations, performs health checks. Supporting: Redis for caching, Dex for SSO, Notification controller for alerts.

**Q3: How do you handle secrets in a GitOps workflow?**

**A:** Secrets cannot be stored in Git in plain text. Solutions: 1) Sealed Secrets — encrypt secrets client-side, store encrypted version in Git, controller decrypts in-cluster. 2) External Secrets Operator — store secrets in external vaults (AWS SM, Azure KV), ESO syncs them to K8s secrets. 3) SOPS — encrypt secret files with KMS keys, decrypt during ArgoCD sync via helm-secrets plugin. 4) Vault + ArgoCD Vault Plugin — reference Vault paths in manifests, plugin resolves at sync time. Best practice: External Secrets Operator for most cases (clean separation, cloud-native).

**Q4: What are ApplicationSets and when would you use them?**

**A:** ApplicationSets automate the generation of ArgoCD Applications using generators. Use when you have: Multiple clusters (cluster generator creates apps per cluster), multiple environments from same source (list generator), dynamic app discovery (git generator discovers new apps from directory structure), or PR-based preview environments (pull request generator). They reduce duplication and enable self-service — adding a directory or cluster automatically creates the ArgoCD Application.

**Q5: How do you implement promotion between environments with GitOps?**

**A:** Options: 1) Branch-based — dev branch → staging branch → main (prod). Merge = promote. 2) Directory-based — overlays/dev, overlays/staging, overlays/prod. Update each overlay's image tag. 3) PR-based promotion — CI creates PR to update production overlay, requires approval to merge. Best practice: PR-based with directory overlays. CI auto-deploys to dev/staging, creates PR for production. PR approval = deployment approval. ArgoCD syncs automatically after merge.

**Q6: How does ArgoCD detect and handle drift?**

**A:** ArgoCD continuously compares the live cluster state against the desired state in Git (reconciliation loop, default every 3 minutes). If someone manually changes a resource (kubectl edit), ArgoCD detects the drift and marks the app as "OutOfSync." With selfHeal enabled, ArgoCD automatically reverts the manual change to match Git. Without selfHeal, it only reports the drift. You can configure ignoreDifferences for fields that legitimately change (e.g., HPA replicas, annotation timestamps).

**Q7: Describe a complete GitOps CI/CD workflow from code commit to production.**

**A:** 1) Developer pushes code to app repo. 2) CI triggers: lint, test, SAST scan, build container image, container scan, push to registry, sign image. 3) CI updates config repo: changes image tag in staging overlay, commits and pushes. 4) ArgoCD (staging) detects config change, syncs new image to staging cluster. 5) Automated tests run against staging. 6) On success, CI creates PR updating production overlay. 7) Team reviews and approves PR. 8) PR merge triggers ArgoCD (production) sync. 9) ArgoCD deploys, health checks pass. 10) Monitoring confirms healthy deployment.

**Q8: How do you monitor and troubleshoot ArgoCD?**

**A:** Monitoring: Expose ArgoCD metrics to Prometheus (ServiceMonitor), alert on key metrics — argocd_app_sync_status (OutOfSync), argocd_app_health_status (Degraded), git request failures, sync duration. Troubleshooting: Check app sync status (argocd app get <app>), view sync history and errors, check repo-server logs for manifest generation issues, check application-controller logs for sync failures, verify Git connectivity and credentials, use `argocd app diff` to see what would change.

---

## Architecture/Design Questions (7)

**Q1: Design a highly available deployment for a critical banking application in Saudi Arabia.**

**A:** Requirements: SAMA compliance, data residency (KSA), HA, DR. Architecture: Multi-AZ EKS in AWS Bahrain (me-south-1), RDS Multi-AZ for database, ALB with WAF for edge protection. Security: mTLS between services, Vault for secrets, Network Policies, Pod Security (restricted), full audit logging (retained 5 years). DR: Cross-region to another GCC region with data replication, RPO < 1 hour, RTO < 4 hours. GitOps with ArgoCD for deployments, comprehensive monitoring with 24/7 alerting. All traffic encrypted, all access logged, MFA everywhere.

**Q2: How would you migrate a monolithic application to microservices using DevOps practices?**

**A:** Strangler fig pattern: 1) Containerize the monolith first (lift and shift). 2) Set up CI/CD pipeline for the monolith. 3) Identify bounded contexts for extraction. 4) Extract one service at a time, starting with low-risk, high-value services. 5) Use API gateway for routing (gradually shift traffic). 6) Each new service gets: its own repo, pipeline, container, deployment. 7) Implement service mesh for inter-service communication. 8) Add comprehensive monitoring per service. Timeline: 12-18 months for a medium-complexity monolith.

**Q3: Design a multi-tenant Kubernetes platform for different development teams.**

**A:** Isolation layers: Namespace per team/environment, Network Policies (deny cross-namespace), Resource Quotas and LimitRanges per namespace, separate node pools for sensitive workloads. Access control: RBAC — team members get namespace-scoped roles only, ArgoCD projects restrict team access to their namespaces. Security: Pod Security Standards (baseline minimum), OPA/Gatekeeper policies (no privileged containers, required labels, approved registries). Monitoring: Per-namespace dashboards, cost allocation by namespace. Self-service: Teams manage their own namespaces through GitOps.

**Q4: How would you design a disaster recovery strategy for a Kubernetes-based platform?**

**A:** Tiers: Tier 1 (critical) — RPO: 5 min, RTO: 15 min. Active-active across AZs, database replication. Tier 2 (important) — RPO: 1 hour, RTO: 4 hours. Standby cluster in another AZ, automated failover. Tier 3 (standard) — RPO: 24 hours, RTO: 8 hours. Backup and restore. Implementation: Velero for K8s resource backup, database-native replication for data, GitOps makes cluster rebuild trivial (ArgoCD redeploys everything from Git), external DNS for traffic switching, regular DR drills (quarterly minimum).

**Q5: How do you optimize a CI/CD pipeline that takes 45 minutes?**

**A:** Analysis: Identify bottlenecks (build, test, scanning). Optimizations: 1) Parallelize — run SAST, unit tests, and lint in parallel. 2) Caching — Docker layer cache, dependency cache (npm/pip), Terraform plugin cache. 3) Incremental — only test changed modules, skip unchanged container builds. 4) Faster tools — replace tools with faster alternatives (Semgrep over SonarQube for SAST). 5) Runner optimization — larger runners, ARM-based runners for builds. 6) Split pipeline — fast checks gate merge, slow checks (DAST, full scan) run post-merge. Target: < 15 minutes for developer feedback loop.

**Q6: Design a secrets management strategy for a company with 100+ microservices.**

**A:** Central vault: HashiCorp Vault (or cloud-native equivalent) as single source of truth. Access pattern: Each service has its own Vault policy (least privilege), authenticated via Kubernetes service account (K8s auth method). Secret types: Static secrets in KV store with rotation, dynamic secrets for databases (auto-generated, auto-expired), PKI engine for TLS certificates. Integration: External Secrets Operator in each cluster syncs to K8s secrets, Vault Agent sidecar for apps needing direct Vault access. Operations: Centralized audit logging, automated rotation, break-glass procedures documented.

**Q7: How would you implement a platform engineering team's internal developer platform?**

**A:** Core components: 1) Self-service portal — developers request environments via Git PR. 2) Standardized templates — Backstage software templates for new services (Dockerfile, CI pipeline, K8s manifests). 3) GitOps foundation — ArgoCD manages all deployments, developers only interact with their config repo. 4) Guardrails — OPA policies enforce standards, resource quotas prevent abuse. 5) Observability — pre-configured dashboards per service, standardized logging format. 6) Documentation — developer portal (Backstage) with API docs, runbooks, architecture diagrams. Goal: Developers focus on code, platform handles everything else.

---

# SECTION 8: FREE RESOURCES WITH LINKS

---

| # | Resource | URL | Topic |
|---|----------|-----|-------|
| 1 | OWASP Top 10 | https://owasp.org/www-project-top-ten/ | Application Security |
| 2 | ArgoCD Documentation | https://argo-cd.readthedocs.io/ | GitOps |
| 3 | Kubernetes Security Best Practices | https://kubernetes.io/docs/concepts/security/ | K8s Security |
| 4 | HashiCorp Vault Learn | https://developer.hashicorp.com/vault/tutorials | Secret Management |
| 5 | CNCF Security Whitepaper | https://github.com/cncf/tag-security/blob/main/security-whitepaper/v2/cloud-native-security-whitepaper.md | Cloud Native Security |
| 6 | Trivy Documentation | https://aquasecurity.github.io/trivy/ | Container Scanning |
| 7 | GitOps Working Group | https://opengitops.dev/ | GitOps Principles |
| 8 | KodeKloud Free Labs | https://kodekloud.com/courses/labs/ | Hands-on Practice |
| 9 | CKS Exam Curriculum | https://github.com/cncf/curriculum/blob/master/CKS_Curriculum.md | K8s Security Cert |
| 10 | SAMA Cyber Security Framework | https://www.sama.gov.sa/en-US/RulesInstructions/ | GCC Compliance |
| 11 | Semgrep Rules Registry | https://semgrep.dev/explore | SAST Rules |
| 12 | Cosign/Sigstore Docs | https://docs.sigstore.dev/ | Image Signing |
| 13 | External Secrets Operator | https://external-secrets.io/ | Secret Management |
| 14 | Falco Documentation | https://falco.org/docs/ | Runtime Security |
| 15 | DevSecOps Playbook | https://dodcio.defense.gov/Portals/0/Documents/Library/DevSecOpsPlaybook.pdf | DevSecOps Framework |
| 16 | Kustomize Documentation | https://kustomize.io/ | K8s Config Management |
| 17 | OPA/Gatekeeper | https://open-policy-agent.github.io/gatekeeper/ | Policy Enforcement |
| 18 | NIST Cybersecurity Framework | https://www.nist.gov/cyberframework | Security Framework |

---

# SECTION 9: COMPLETE 6-MONTH PROGRAM SUMMARY

---

## What You Should Know After 6 Months

```
Month 1: Linux, Networking & Cloud Fundamentals
├── Linux administration, scripting (Bash)
├── Networking (TCP/IP, DNS, load balancing)
└── Cloud basics (AWS/Azure core services, IAM)

Month 2: Infrastructure as Code
├── Terraform (modules, state, workspaces)
├── Configuration management (Ansible basics)
└── Cloud architecture patterns

Month 3: Containers & Kubernetes
├── Docker (building, optimizing, security)
├── Kubernetes (deployments, services, storage)
└── Helm charts, Kustomize

Month 4: CI/CD Pipelines
├── GitLab CI / Azure DevOps pipelines
├── Pipeline design patterns
├── Testing automation
└── Artifact management

Month 5: Monitoring & Observability
├── Prometheus + Grafana
├── Log management (ELK/Loki)
├── Alerting strategies
└── SRE principles (SLIs, SLOs, error budgets)

Month 6: Security, GitOps & Capstone (THIS MONTH)
├── DevSecOps (SAST, DAST, SCA, container scanning)
├── Kubernetes security (RBAC, Network Policies, secrets)
├── GitOps with ArgoCD
├── Compliance (SAMA, NCA, PDPL)
└── Capstone project (all skills combined)
```

## Certification Roadmap

| Priority | Certification | Difficulty | Value in GCC |
|----------|--------------|------------|--------------|
| 1 | AWS Solutions Architect Associate | Medium | Very High |
| 2 | CKA (Certified Kubernetes Administrator) | Medium-Hard | Very High |
| 3 | Terraform Associate | Medium | High |
| 4 | AZ-104 (Azure Administrator) | Medium | Very High |
| 5 | CKS (Certified Kubernetes Security) | Hard | High |
| 6 | AWS DevOps Professional | Hard | Very High |
| 7 | CKAD (Certified Kubernetes App Developer) | Medium | Medium |

**Recommended first year:** AWS SAA + CKA + Terraform Associate = strongest DevOps foundation

## Next Steps

1. **Complete the capstone project** — This is your portfolio piece for interviews
2. **Get certified** — Start with AWS SAA or CKA (most recognized in GCC)
3. **Build in public** — Blog about your learning journey, share on LinkedIn
4. **Contribute to open source** — Even small PRs show initiative and skill
5. **Network** — Attend GCC tech meetups, CNCF community days, AWS/Azure summits
6. **Apply broadly** — Target companies doing cloud transformation in GCC (banking, telecom, government)
7. **Keep learning** — DevOps evolves rapidly. Dedicate 5-10 hours/week to staying current
8. **Specialize** — After broad foundation, go deep in one area (security, platform engineering, or SRE)

---

**Congratulations on completing the 6-month DevOps Upskilling Program!** 🎉

You now have the knowledge foundation to work as a DevOps Engineer in the GCC market. The capstone project demonstrates your ability to combine all these skills into a production-ready solution. Keep building, keep learning, and remember: DevOps is a journey, not a destination.

---
*Document generated for the DevOps Upskilling Program — GCC Market Focus*
*Last updated: July 2026*
