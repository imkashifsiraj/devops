# Month 2 Part 2: Kubernetes (K8s) — Complete Study Guide

> **DevOps Upskilling Program — GCC Job Market Focus**
> Master container orchestration with Kubernetes for production-grade deployments.

---

## 1. What is Kubernetes? Why Container Orchestration?

**Kubernetes** (K8s) is an open-source container orchestration platform originally developed by Google, now maintained by the CNCF. It automates deploying, scaling, and managing containerized applications.

### Why Container Orchestration?

| Challenge | How K8s Solves It |
|-----------|-------------------|
| Running containers across multiple hosts | Cluster management with scheduler |
| Service discovery | Built-in DNS and Service resources |
| Load balancing | Services and Ingress |
| Self-healing | Restart policies, health checks |
| Scaling | HPA, VPA, Cluster Autoscaler |
| Rolling updates with zero downtime | Deployment strategies |
| Secret/config management | ConfigMaps and Secrets |
| Storage orchestration | PV, PVC, StorageClasses |

### Key Benefits
- **Declarative configuration** — define desired state, K8s maintains it
- **Self-healing** — automatically restarts failed containers, replaces pods
- **Horizontal scaling** — scale up/down based on metrics
- **Service discovery & load balancing** — built-in networking
- **Automated rollouts & rollbacks** — zero-downtime deployments
- **Multi-cloud & hybrid** — runs anywhere (AWS EKS, Azure AKS, GCP GKE, on-prem)

---

## 2. Kubernetes Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                             │
│  ┌───────────┐  ┌──────┐  ┌───────────┐  ┌──────────────────┐  │
│  │ API Server│  │ etcd │  │ Scheduler │  │Controller Manager│  │
│  └───────────┘  └──────┘  └───────────┘  └──────────────────┘  │
│  ┌──────────────────────┐                                       │
│  │Cloud Controller Mgr  │                                       │
│  └──────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
        │                    │                    │
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  WORKER NODE  │  │  WORKER NODE  │  │  WORKER NODE  │
│ ┌───────────┐ │  │ ┌───────────┐ │  │ ┌───────────┐ │
│ │  kubelet  │ │  │ │  kubelet  │ │  │ │  kubelet  │ │
│ │kube-proxy │ │  │ │kube-proxy │ │  │ │kube-proxy │ │
│ │ container │ │  │ │ container │ │  │ │ container │ │
│ │  runtime  │ │  │ │  runtime  │ │  │ │  runtime  │ │
│ └───────────┘ │  │ └───────────┘ │  │ └───────────┘ │
│   [Pods...]   │  │   [Pods...]   │  │   [Pods...]   │
└───────────────┘  └───────────────┘  └───────────────┘
```

### Control Plane Components

#### API Server (`kube-apiserver`)
- **Front door** to the cluster — all communication goes through it
- RESTful API that validates and processes requests
- Only component that talks directly to etcd
- Handles authentication, authorization (RBAC), admission control
- Stateless — can run multiple instances for HA

#### etcd
- Distributed key-value store holding **all cluster state**
- Stores resource definitions, secrets, configs, cluster membership
- Uses Raft consensus for consistency
- Critical to backup regularly: `etcdctl snapshot save backup.db`
- Typically 3 or 5 nodes for high availability

#### Scheduler (`kube-scheduler`)
- Watches for unscheduled pods (pods with no node assigned)
- Selects optimal node based on: resource requests, affinity/anti-affinity, taints/tolerations, topology
- Two phases: **filtering** (which nodes can run it) → **scoring** (which is best)

#### Controller Manager (`kube-controller-manager`)
- Runs controller loops that reconcile current state → desired state
- Key controllers: ReplicaSet, Deployment, Node, Job, ServiceAccount, Endpoint
- Each controller watches the API server and takes corrective action

#### Cloud Controller Manager
- Integrates with cloud provider APIs (AWS, Azure, GCP)
- Manages: load balancers, routes, node lifecycle, storage volumes
- Separated from core controller manager for portability

### Worker Node Components

#### kubelet
- Agent running on every worker node
- Registers node with the cluster
- Receives PodSpecs from API server, ensures containers are running
- Reports node and pod status back to control plane
- Manages liveness/readiness probes

#### kube-proxy
- Network proxy on each node
- Implements Service abstraction (virtual IPs)
- Modes: iptables (default), IPVS (better performance at scale), userspace (legacy)
- Maintains network rules for pod-to-service communication

#### Container Runtime
- Software that runs containers (CRI-compliant)
- Options: **containerd** (most common), CRI-O, Docker (deprecated as runtime in 1.24+)
- Pulls images, starts/stops containers, manages container lifecycle

### Communication Flow
1. User → `kubectl` → API Server (HTTPS, authenticated)
2. API Server → etcd (store/retrieve state)
3. Scheduler → API Server (watch unscheduled pods, assign nodes)
4. Controller Manager → API Server (watch resources, reconcile)
5. kubelet → API Server (watch assigned pods, report status)
6. kube-proxy → API Server (watch Services/Endpoints, update rules)

---

## 3. Core Resources

### Pods

The smallest deployable unit. A pod wraps one or more containers sharing network and storage.

#### Basic Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
      resources:
        requests:
          memory: "64Mi"
          cpu: "100m"
        limits:
          memory: "128Mi"
          cpu: "250m"
```

#### Multi-Container Pod (Sidecar Pattern)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
    - name: app
      image: myapp:1.0
      ports:
        - containerPort: 8080
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
    - name: log-shipper
      image: fluentd:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
          readOnly: true
  volumes:
    - name: shared-logs
      emptyDir: {}
```

#### Init Containers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
    - name: wait-for-db
      image: busybox:1.36
      command: ['sh', '-c', 'until nc -z mysql-svc 3306; do echo waiting for db; sleep 2; done']
    - name: migrate-db
      image: myapp:1.0
      command: ['python', 'manage.py', 'migrate']
  containers:
    - name: app
      image: myapp:1.0
      ports:
        - containerPort: 8080
```

### ReplicaSets

Ensures a specified number of pod replicas are running. Rarely created directly — use Deployments.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

### Deployments

The standard way to manage stateless applications. Provides rolling updates, rollbacks, and scaling.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # max pods above desired during update
      maxUnavailable: 0    # zero downtime
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: webapp:2.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "128Mi"
              cpu: "200m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

**Deployment commands:**
```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Check rollout status
kubectl rollout status deployment/webapp

# View rollout history
kubectl rollout history deployment/webapp

# Rollback to previous version
kubectl rollout undo deployment/webapp

# Rollback to specific revision
kubectl rollout undo deployment/webapp --to-revision=2

# Scale deployment
kubectl scale deployment/webapp --replicas=5

# Update image (imperative)
kubectl set image deployment/webapp webapp=webapp:3.0
```

### Services

Services provide stable networking to a set of pods selected by labels.

#### ClusterIP (default — internal only)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80          # service port
      targetPort: 8080  # container port
      protocol: TCP
```

#### NodePort (accessible from outside on node IP:port)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080   # range 30000-32767
```

#### LoadBalancer (cloud provider external LB)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
    - port: 443
      targetPort: 8080
      protocol: TCP
```

#### ExternalName (CNAME alias to external service)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.example.com
```

#### Headless Service (no ClusterIP — for StatefulSets)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
```

### Namespaces

Logical isolation within a cluster. Use for environments, teams, or applications.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    env: production
```

```bash
# Create namespace
kubectl create namespace staging

# List namespaces
kubectl get namespaces

# Set default namespace for context
kubectl config set-context --current --namespace=production

# Get resources in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A
```

### ConfigMaps

Store non-sensitive configuration data as key-value pairs.

#### From literal values

```bash
kubectl create configmap app-config \
  --from-literal=DB_HOST=mysql-svc \
  --from-literal=DB_PORT=3306 \
  --from-literal=LOG_LEVEL=info
```

#### From file

```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```

#### YAML definition

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: mysql-svc
  DB_PORT: "3306"
  LOG_LEVEL: info
  app.properties: |
    server.port=8080
    spring.profiles.active=production
    cache.ttl=300
```

#### Using ConfigMap in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
    - name: app
      image: myapp:1.0
      # As environment variables
      envFrom:
        - configMapRef:
            name: app-config
      # Or specific keys
      env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_HOST
      # As mounted file
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

### Secrets

Store sensitive data (passwords, tokens, keys). Base64-encoded (NOT encrypted by default!).

#### Types of Secrets
| Type | Usage |
|------|-------|
| `Opaque` | Generic, user-defined data |
| `kubernetes.io/tls` | TLS certificates |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials |
| `kubernetes.io/service-account-token` | Service account tokens |

#### Create Secrets

```bash
# From literal
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password='S3cur3P@ss!'

# From file (e.g., TLS)
kubectl create secret tls my-tls \
  --cert=tls.crt --key=tls.key

# Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass
```

#### YAML definition

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
type: Opaque
data:
  username: YWRtaW4=          # echo -n 'admin' | base64
  password: UzNjdXIzUEBzcyE=  # echo -n 'S3cur3P@ss!' | base64
---
# Using stringData (plain text, auto-encoded)
apiVersion: v1
kind: Secret
metadata:
  name: db-creds-plain
type: Opaque
stringData:
  username: admin
  password: S3cur3P@ss!
```

#### Using Secrets in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-secrets
spec:
  containers:
    - name: app
      image: myapp:1.0
      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-creds
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-creds
              key: password
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: db-creds
```

> **⚠️ Security Concerns:** Secrets are base64-encoded, not encrypted. Enable encryption at rest via `EncryptionConfiguration`. Use external secret managers (AWS Secrets Manager, HashiCorp Vault) with operators like External Secrets Operator for production.

### DaemonSets

Ensures a pod runs on **every** node (or selected nodes). Used for log collectors, monitoring agents, network plugins.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          effect: NoSchedule
      containers:
        - name: node-exporter
          image: prom/node-exporter:v1.7.0
          ports:
            - containerPort: 9100
              hostPort: 9100
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 100m
              memory: 128Mi
```

### StatefulSets

For stateful applications needing stable identities, persistent storage, and ordered deployment.

**When to use StatefulSet vs Deployment:**
| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod identity | Random names | Ordered: app-0, app-1, app-2 |
| Storage | Shared or ephemeral | Dedicated PVC per pod |
| Scaling | Parallel | Ordered (one at a time) |
| Use case | Stateless apps | Databases, Kafka, ZooKeeper |

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless  # must reference headless service
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: root-password
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: gp3
        resources:
          requests:
            storage: 20Gi
```

### Jobs and CronJobs

#### Job (run-to-completion task)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 300
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: myapp:1.0
          command: ["python", "manage.py", "migrate"]
```

#### CronJob (scheduled tasks)

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: backup-tool:1.0
              command: ["/bin/sh", "-c", "pg_dump $DB_URL > /backups/$(date +%Y%m%d).sql"]
              envFrom:
                - secretRef:
                    name: db-creds
```

---

## 4. Networking

### Pod-to-Pod Communication
- Every pod gets its own IP address
- Pods can communicate directly without NAT (flat network)
- All containers in a pod share the same network namespace (localhost)
- CNI plugin handles IP assignment and routing between nodes

### Service Networking and kube-proxy Modes

| Mode | Description | Best For |
|------|-------------|----------|
| **iptables** | Default. Creates iptables rules for each service | Small-medium clusters |
| **IPVS** | Uses Linux IPVS (kernel-level LB) | Large clusters (>1000 services) |
| **userspace** | Legacy, proxies in userspace | Not recommended |

### DNS in Kubernetes (CoreDNS)

Every service gets a DNS entry:
```
<service-name>.<namespace>.svc.cluster.local
```

Examples:
```bash
# Service in same namespace
curl http://backend-svc

# Service in different namespace
curl http://backend-svc.production.svc.cluster.local

# Headless service pod DNS (StatefulSet)
mysql-0.mysql-headless.default.svc.cluster.local
```

### NetworkPolicies

Control traffic flow between pods. By default, all pods can talk to all pods.

#### Deny all ingress to a namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}   # applies to all pods
  policyTypes:
    - Ingress
```

#### Allow specific ingress

```yaml
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
```

#### Egress restriction

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
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
              app: database
      ports:
        - protocol: TCP
          port: 5432
    - to:  # allow DNS
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
```

### CNI Plugins

| Plugin | Features | Best For |
|--------|----------|----------|
| **Calico** | NetworkPolicy, BGP routing, eBPF | Production, hybrid clouds |
| **Cilium** | eBPF-based, L7 policies, observability | Advanced security, service mesh |
| **Flannel** | Simple overlay (VXLAN) | Development, simple setups |
| **AWS VPC CNI** | Native VPC networking, pod IPs from VPC | EKS clusters |

---

## 5. Storage

### Volumes, PersistentVolumes, PersistentVolumeClaims

#### EmptyDir Volume (ephemeral, pod lifetime)

```yaml
volumes:
  - name: cache
    emptyDir:
      sizeLimit: 500Mi
```

#### PersistentVolume (admin provisions)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 50Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: nfs.example.com
    path: /exports/data
```

#### PersistentVolumeClaim (developer requests)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp3
  resources:
    requests:
      storage: 10Gi
```

#### Using PVC in a Pod

```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: data
          mountPath: /app/data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: app-storage
```

### StorageClasses and Dynamic Provisioning

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

### Access Modes

| Mode | Abbrev | Description |
|------|--------|-------------|
| ReadWriteOnce | RWO | Single node read-write |
| ReadOnlyMany | ROX | Multiple nodes read-only |
| ReadWriteMany | RWX | Multiple nodes read-write (NFS, EFS) |


---

## 6. Ingress

### Ingress vs LoadBalancer Service
- **LoadBalancer Service**: One cloud LB per service (expensive)
- **Ingress**: Single LB routing to multiple services based on rules (cost-effective)

### Ingress Controller (Nginx)

Install with Helm:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

### Path-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### Host-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
    - host: dashboard.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: dashboard-service
                port:
                  number: 80
```

### TLS Termination

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - app.example.com
      secretName: app-tls-secret
  rules:
    - host: app.example.com
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

### Useful Annotations

```yaml
annotations:
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
  nginx.ingress.kubernetes.io/proxy-body-size: "50m"
  nginx.ingress.kubernetes.io/rate-limit: "10"
  nginx.ingress.kubernetes.io/cors-allow-origin: "*"
  nginx.ingress.kubernetes.io/affinity: "cookie"
  nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
```

---

## 7. Scaling

### HorizontalPodAutoscaler (HPA)

Scales pods based on observed metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
```

```bash
# Create HPA imperatively
kubectl autoscale deployment webapp --cpu-percent=70 --min=2 --max=20

# Check HPA status
kubectl get hpa
```

> **Prerequisite:** Metrics Server must be installed for HPA to work.

### VerticalPodAutoscaler (VPA)

Adjusts resource requests/limits automatically. Cannot run with HPA on the same metric.

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: webapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  updatePolicy:
    updateMode: Auto  # Off, Initial, Auto
  resourcePolicy:
    containerPolicies:
      - containerName: webapp
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 2
          memory: 4Gi
```

### Cluster Autoscaler

Adjusts the number of nodes in a cluster based on pending pods and resource utilization.

```bash
# Typically deployed as a Deployment
# Configuration varies by cloud provider (EKS, AKS, GKE)
# Key flags:
# --scale-down-unneeded-time=10m
# --scale-down-utilization-threshold=0.5
# --max-node-provision-time=15m
```

### Pod Disruption Budgets (PDB)

Protect availability during voluntary disruptions (node drain, upgrades).

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: webapp-pdb
spec:
  minAvailable: 2        # OR use maxUnavailable: 1
  selector:
    matchLabels:
      app: webapp
```

---

## 8. Health Probes

### Liveness Probe
Restarts container if it fails. Use for deadlock detection.

### Readiness Probe
Removes pod from Service endpoints if it fails. Use for startup dependencies.

### Startup Probe
Disables liveness/readiness until it succeeds. Use for slow-starting applications.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probes-example
spec:
  containers:
    - name: app
      image: myapp:1.0
      ports:
        - containerPort: 8080
      # HTTP probe
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 10
        timeoutSeconds: 3
        failureThreshold: 3
      # HTTP readiness
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
        successThreshold: 1
        failureThreshold: 3
      # Startup probe for slow apps
      startupProbe:
        httpGet:
          path: /healthz
          port: 8080
        failureThreshold: 30
        periodSeconds: 10    # 30 x 10 = 300s max startup time
```

#### TCP Probe

```yaml
livenessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 15
  periodSeconds: 10
```

#### Exec Probe

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

### When to Use Which

| Probe | Purpose | Consequence of Failure |
|-------|---------|----------------------|
| Liveness | Is the app alive? | Container restart |
| Readiness | Can it serve traffic? | Removed from endpoints |
| Startup | Has it finished starting? | Delays other probes |

### Common Mistakes
- Setting `initialDelaySeconds` too short → premature restart loop
- Liveness probe checking dependencies (DB) → cascading restarts
- Same endpoint for liveness and readiness → pod killed when it should just stop receiving traffic
- Not setting `timeoutSeconds` → default 1s may be too short for slow endpoints

---

## 9. RBAC (Role-Based Access Control)

### Concepts
- **Role** — permissions within a namespace
- **ClusterRole** — cluster-wide permissions
- **RoleBinding** — grants Role to a user/group/SA in a namespace
- **ClusterRoleBinding** — grants ClusterRole cluster-wide

### Role (namespace-scoped)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: development
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]
```

### ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list"]
```

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-dev
  namespace: development
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
  - kind: ServiceAccount
    name: ci-bot
    namespace: development
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
  - kind: Group
    name: platform-team
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

### ServiceAccounts

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: production
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/app-role
```

```yaml
# Use in Pod spec
spec:
  serviceAccountName: app-sa
  automountServiceAccountToken: false  # disable if not needed
```

```bash
# Check permissions
kubectl auth can-i create deployments --as=jane --namespace=development
kubectl auth can-i '*' '*' --as=system:serviceaccount:production:app-sa
```


---

## 10. kubectl Commands — Complete Reference

### Essential Commands

```bash
# --- GET RESOURCES ---
kubectl get pods                          # list pods in current namespace
kubectl get pods -A                       # all namespaces
kubectl get pods -o wide                  # show node, IP
kubectl get pods -o yaml                  # full YAML output
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods -l app=nginx             # filter by label
kubectl get pods --field-selector=status.phase=Running
kubectl get all                           # pods, services, deployments, etc.

# --- DESCRIBE (detailed info + events) ---
kubectl describe pod nginx-pod
kubectl describe node worker-1
kubectl describe service backend-svc

# --- LOGS ---
kubectl logs pod-name
kubectl logs pod-name -c container-name   # specific container
kubectl logs pod-name --previous          # previous crashed container
kubectl logs -f pod-name                  # follow/stream
kubectl logs -l app=nginx --all-containers  # by label
kubectl logs pod-name --tail=100          # last 100 lines
kubectl logs pod-name --since=1h          # last hour

# --- EXEC (run commands in container) ---
kubectl exec -it pod-name -- /bin/bash
kubectl exec pod-name -- ls /app
kubectl exec -it pod-name -c sidecar -- sh

# --- APPLY / CREATE / DELETE ---
kubectl apply -f manifest.yaml
kubectl apply -f ./manifests/             # apply directory
kubectl apply -f https://url/manifest.yaml
kubectl create -f manifest.yaml           # error if exists
kubectl delete -f manifest.yaml
kubectl delete pod nginx-pod
kubectl delete pods -l app=test           # delete by label
kubectl delete namespace staging          # delete entire namespace

# --- EDIT (opens in editor) ---
kubectl edit deployment webapp
kubectl edit svc backend-svc

# --- PORT-FORWARD ---
kubectl port-forward pod/nginx-pod 8080:80
kubectl port-forward svc/backend-svc 3000:80
kubectl port-forward deployment/webapp 8080:8080

# --- TOP (resource usage — requires metrics-server) ---
kubectl top nodes
kubectl top pods
kubectl top pods --sort-by=memory

# --- ROLLOUT ---
kubectl rollout status deployment/webapp
kubectl rollout history deployment/webapp
kubectl rollout undo deployment/webapp
kubectl rollout undo deployment/webapp --to-revision=3
kubectl rollout restart deployment/webapp   # trigger rolling restart
kubectl rollout pause deployment/webapp
kubectl rollout resume deployment/webapp

# --- SCALE ---
kubectl scale deployment webapp --replicas=5
kubectl scale statefulset mysql --replicas=3

# --- LABELS & ANNOTATIONS ---
kubectl label pod nginx-pod env=production
kubectl label pod nginx-pod env-                  # remove label
kubectl annotate deployment webapp note="hotfix applied"

# --- CONFIG (context management) ---
kubectl config get-contexts
kubectl config current-context
kubectl config use-context production-cluster
kubectl config set-context --current --namespace=staging
kubectl config view --minify                      # current context only

# --- OTHER USEFUL ---
kubectl explain pod.spec.containers              # API docs in terminal
kubectl api-resources                            # list all resource types
kubectl get events --sort-by=.lastTimestamp
kubectl diff -f manifest.yaml                    # preview changes
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
kubectl cordon node-1                            # mark unschedulable
kubectl uncordon node-1                          # mark schedulable
kubectl taint nodes node-1 key=value:NoSchedule
```

### Useful Flags

| Flag | Description |
|------|-------------|
| `-o yaml` | Output as YAML |
| `-o json` | Output as JSON |
| `-o wide` | Additional columns (node, IP) |
| `-o name` | Only resource names |
| `--watch` / `-w` | Watch for changes |
| `--sort-by=.status.startTime` | Sort output |
| `--dry-run=client -o yaml` | Generate YAML without applying |
| `-n <namespace>` | Specify namespace |
| `--all-namespaces` / `-A` | All namespaces |
| `--selector` / `-l` | Label selector |

```bash
# Generate YAML template without creating
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment webapp --image=webapp:1.0 --dry-run=client -o yaml > deploy.yaml
kubectl expose deployment webapp --port=80 --dry-run=client -o yaml > svc.yaml
```

---

## 11. Troubleshooting Kubernetes

### Pod Stuck in Pending

**Causes & Solutions:**
```bash
kubectl describe pod <pod-name>  # check Events section

# Common causes:
# 1. Insufficient resources → scale cluster or reduce requests
# 2. No matching node (nodeSelector, affinity) → check labels
# 3. PVC not bound → check PV/StorageClass
# 4. Taints without tolerations → add toleration

# Debug:
kubectl get nodes -o wide
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
kubectl get pvc
```

### CrashLoopBackOff

**Pod starts but keeps crashing.**
```bash
# Check logs from crashed container
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>  # check exit code

# Common causes:
# Exit code 1: Application error (check logs)
# Exit code 137: OOMKilled (increase memory limit)
# Exit code 139: Segfault
# Missing config/secrets, wrong command, missing dependencies

# Debug:
kubectl run debug --image=busybox --rm -it -- sh  # test from inside cluster
```

### ImagePullBackOff

```bash
kubectl describe pod <pod-name>  # check image name, registry

# Common causes:
# 1. Wrong image name/tag → verify in registry
# 2. Private registry without imagePullSecrets
# 3. Registry rate limits (Docker Hub)

# Fix private registry:
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user --docker-password=pass

# Add to pod spec:
spec:
  imagePullSecrets:
    - name: regcred
```

### OOMKilled

```bash
kubectl describe pod <pod-name> | grep -i oom
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].lastState}'

# Solutions:
# 1. Increase memory limits
# 2. Fix memory leak in application
# 3. Use VPA to find correct values
kubectl top pod <pod-name>  # check actual usage
```

### Service Not Accessible

```bash
# Verify service exists and has endpoints
kubectl get svc <service-name>
kubectl get endpoints <service-name>

# Common causes:
# 1. No endpoints → labels don't match between service and pods
kubectl get pods --show-labels
# 2. Wrong port/targetPort
# 3. Pod not ready (failing readiness probe)

# Test from inside cluster:
kubectl run test --image=busybox --rm -it -- wget -qO- http://service-name:port
```

### DNS Resolution Issues

```bash
# Test DNS from inside a pod
kubectl run dns-test --image=busybox:1.36 --rm -it -- nslookup kubernetes.default
kubectl run dns-test --image=busybox:1.36 --rm -it -- nslookup <service-name>.<namespace>.svc.cluster.local

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Check resolv.conf in pod
kubectl exec <pod> -- cat /etc/resolv.conf
```

### Node NotReady

```bash
kubectl get nodes
kubectl describe node <node-name>

# Check conditions: MemoryPressure, DiskPressure, PIDPressure
# Check kubelet:
ssh node-1 "systemctl status kubelet"
ssh node-1 "journalctl -u kubelet --since '10 minutes ago'"
```

### Debugging Methodology

1. **Identify** — `kubectl get pods` (what's unhealthy?)
2. **Describe** — `kubectl describe pod/svc/node` (events tell the story)
3. **Logs** — `kubectl logs` (application errors)
4. **Exec** — `kubectl exec -it` (test from inside)
5. **Network** — test DNS, connectivity from debug pod
6. **Cluster** — check nodes, resources, control plane components

---

## 12. Helm Basics

### What is Helm?
Helm is the **package manager for Kubernetes**. It packages K8s manifests into reusable **charts** with templating and versioning.

### Chart Structure

```
mychart/
├── Chart.yaml          # Chart metadata (name, version, dependencies)
├── values.yaml         # Default configuration values
├── charts/             # Dependency charts
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Template helpers
│   └── NOTES.txt       # Post-install notes
└── .helmignore
```

### Helm Commands

```bash
# --- REPOSITORY MANAGEMENT ---
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx

# --- INSTALL ---
helm install my-release bitnami/nginx
helm install my-release ./mychart -f custom-values.yaml
helm install my-release bitnami/nginx --namespace web --create-namespace
helm install my-release bitnami/nginx --set service.type=NodePort

# --- UPGRADE ---
helm upgrade my-release bitnami/nginx --set replicaCount=3
helm upgrade my-release ./mychart -f production-values.yaml
helm upgrade --install my-release ./mychart  # install if not exists

# --- ROLLBACK ---
helm rollback my-release 1          # rollback to revision 1
helm history my-release             # view release history

# --- LIST & STATUS ---
helm list                           # list releases
helm list -A                        # all namespaces
helm status my-release

# --- UNINSTALL ---
helm uninstall my-release

# --- TEMPLATE (render without installing) ---
helm template my-release ./mychart -f values.yaml
helm template my-release ./mychart --debug  # show with debug info
```

### Values File Example

```yaml
# values.yaml
replicaCount: 3

image:
  repository: myapp
  tag: "2.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  host: app.example.com

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi

env:
  LOG_LEVEL: info
  DB_HOST: mysql-svc
```

### Template Example

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
            - containerPort: 8080
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- if .Values.env }}
          env:
            {{- range $key, $value := .Values.env }}
            - name: {{ $key }}
              value: {{ $value | quote }}
            {{- end }}
          {{- end }}
```


---

## 13. Production Best Practices

1. **Always set resource requests AND limits** — prevents noisy neighbors, enables autoscaling
2. **Use readiness probes on every container** — prevents traffic to unready pods
3. **Use namespaces** — isolate environments (dev/staging/prod), teams, or applications
4. **Implement NetworkPolicies** — default-deny, then whitelist needed traffic
5. **Use Pod Disruption Budgets** — ensure availability during node maintenance
6. **Run multiple replicas** — minimum 2-3 replicas for any production workload
7. **Use anti-affinity** — spread pods across nodes/zones for HA
8. **Store configs in ConfigMaps, secrets in Secrets** — never hardcode in images
9. **Use external secret management** — AWS Secrets Manager, Vault via External Secrets Operator
10. **Enable RBAC with least privilege** — never use `cluster-admin` for apps
11. **Use `automountServiceAccountToken: false`** unless needed
12. **Set security contexts** — `runAsNonRoot`, `readOnlyRootFilesystem`, drop all capabilities
13. **Pin image tags** — never use `:latest` in production, use SHA digests for critical workloads
14. **Implement pod topology spread constraints** — distribute across failure domains
15. **Use priority classes** — ensure critical workloads get scheduled first
16. **Enable audit logging** — track who did what in your cluster
17. **Limit container images to trusted registries** — use admission controllers (OPA/Kyverno)
18. **Set up monitoring and alerting** — Prometheus + Grafana + alerting rules
19. **Implement GitOps** — ArgoCD or Flux for declarative continuous deployment
20. **Regularly update K8s** — stay within supported versions (N-2 policy)

---

## 14. Lessons Learned / Pro Tips

1. **`kubectl explain` is your best friend** — `kubectl explain pod.spec.containers.resources` gives inline API docs
2. **Use `--dry-run=client -o yaml`** to generate manifests quickly during CKA exams
3. **Labels are the glue** — services, deployments, network policies all use label selectors; inconsistent labels = broken connections
4. **Liveness probes should NOT check dependencies** — if your DB is down, restarting your app won't fix it; use readiness instead
5. **`emptyDir` dies with the pod** — for anything persistent, use PVCs
6. **StatefulSets need headless services** — without `clusterIP: None`, DNS won't resolve individual pods
7. **DNS debugging**: always test with `busybox:1.36` not `latest` (DNS broken in some busybox versions)
8. **Finalizers can block deletion** — if a namespace is stuck in Terminating, check for stuck finalizers
9. **Resource limits on namespaces** — use LimitRange and ResourceQuota to prevent runaway workloads
10. **Init containers run sequentially** — use them for ordering dependencies, not parallel work
11. **Use `kubectl diff`** before `apply` — see what will change before applying
12. **Helm `--atomic` flag** — auto-rollback if upgrade fails, saves you in CI/CD pipelines
13. **Node affinity > nodeSelector** — more expressive and flexible for scheduling constraints
14. **Watch out for `imagePullPolicy: Always`** with mutable tags — can cause slow rollouts and registry load
15. **Set `terminationGracePeriodSeconds`** appropriately — give apps time to drain connections (default 30s)
16. **Use `kubectl get events --sort-by=.lastTimestamp`** — events tell the story of what happened
17. **CIDR overlap kills networking** — ensure pod CIDR, service CIDR, and VPC CIDR don't overlap
18. **Ephemeral containers** (`kubectl debug`) — attach debug containers to running pods without restart

---

## 15. Interview Questions & Answers

### Q1: What happens when you run `kubectl apply -f deployment.yaml`?
**A:** kubectl sends the manifest to the API server → API server validates, authenticates, runs admission controllers → stores in etcd → Deployment controller creates ReplicaSet → ReplicaSet controller creates Pods → Scheduler assigns pods to nodes → kubelet on each node starts containers.

### Q2: How does a Kubernetes Service route traffic to pods?
**A:** Services use label selectors to identify target pods. The Endpoints controller maintains a list of pod IPs. kube-proxy on each node programs iptables/IPVS rules to route traffic to the service's ClusterIP to one of the backend pod IPs using round-robin.

### Q3: What's the difference between a Deployment and a StatefulSet?
**A:** Deployments manage stateless apps with interchangeable pods (random names, shared storage). StatefulSets provide stable network identities (ordered names like pod-0, pod-1), dedicated persistent storage per pod, and ordered deployment/scaling. Use StatefulSets for databases, message brokers, and clustered apps.

### Q4: Explain the difference between liveness and readiness probes.
**A:** Liveness detects if a container is alive — failure triggers a restart. Readiness detects if a container can serve traffic — failure removes it from Service endpoints but doesn't restart it. A pod can be alive but not ready (e.g., loading cache).

### Q5: How does Kubernetes handle rolling updates?
**A:** The Deployment controller creates a new ReplicaSet with the updated spec, gradually scales it up while scaling the old one down, respecting `maxSurge` and `maxUnavailable`. If readiness probes fail on new pods, the rollout pauses. You can rollback with `kubectl rollout undo`.

### Q6: What is a headless service and when would you use it?
**A:** A Service with `clusterIP: None`. Instead of load-balancing, DNS returns individual pod IPs. Used with StatefulSets where clients need to connect to specific pods (e.g., MySQL primary vs replica, Kafka broker partition leaders).

### Q7: Explain Kubernetes RBAC.
**A:** RBAC controls who can do what. Roles/ClusterRoles define permissions (verbs on resources). RoleBindings/ClusterRoleBindings assign those roles to users, groups, or ServiceAccounts. Follow least privilege — give only the permissions needed.

### Q8: How would you debug a pod stuck in Pending?
**A:** Run `kubectl describe pod` to check Events. Common causes: insufficient CPU/memory (check node capacity), unbound PVC (check PVs/StorageClass), node selectors don't match any node, taints without tolerations. Resolution depends on cause — add nodes, fix selectors, or provision storage.

### Q9: What is a PodDisruptionBudget?
**A:** PDBs limit voluntary disruptions (node drain, cluster upgrades) to ensure minimum availability. You specify either `minAvailable` or `maxUnavailable`. Without PDBs, a node drain could kill all pods of a service simultaneously.

### Q10: How does DNS work in Kubernetes?
**A:** CoreDNS runs as pods in kube-system namespace. Every pod gets `/etc/resolv.conf` pointing to CoreDNS ClusterIP. Service DNS format: `service.namespace.svc.cluster.local`. Pod DNS for StatefulSets: `pod-name.service.namespace.svc.cluster.local`.

### Q11: What's the difference between ConfigMap and Secret?
**A:** Both store configuration, but Secrets are base64-encoded and intended for sensitive data. Secrets can be encrypted at rest, have stricter RBAC defaults, and are stored separately in etcd. ConfigMaps are for non-sensitive config like URLs, feature flags, config files.

### Q12: How does the Horizontal Pod Autoscaler work?
**A:** HPA queries the metrics server every 15 seconds (default), compares current metric values to target, calculates desired replicas using: `desired = ceil(current * (currentMetric / targetMetric))`. It scales the Deployment up/down within min/max bounds. Requires Metrics Server.

### Q13: What is a DaemonSet and when would you use it?
**A:** DaemonSet ensures exactly one pod per node (or per selected node). Used for: log collectors (Fluentd/Filebeat), monitoring agents (node-exporter), network plugins (Calico), storage daemons. When nodes are added, DaemonSet automatically schedules a pod on them.

### Q14: Explain the difference between Ingress and a LoadBalancer service.
**A:** LoadBalancer creates one cloud LB per service (expensive, no path routing). Ingress is a single entry point with an Ingress Controller (Nginx, ALB) that routes based on host/path rules to multiple services. Ingress supports TLS termination, rewrites, and rate limiting via annotations.

### Q15: What are init containers?
**A:** Containers that run to completion before app containers start. They run sequentially and must succeed. Use cases: wait for dependencies, run DB migrations, download config files, set filesystem permissions. They have separate resource limits from app containers.

### Q16: How would you perform a zero-downtime deployment?
**A:** Use a Deployment with `strategy: RollingUpdate`, set `maxUnavailable: 0` (never reduce below desired), configure readiness probes (new pods must pass before old ones terminate), set appropriate `terminationGracePeriodSeconds`, use PodDisruptionBudgets, and implement preStop hooks to drain connections.

### Q17: What is a NetworkPolicy?
**A:** A firewall for pod-to-pod traffic. By default all pods can communicate. NetworkPolicies allow you to restrict ingress/egress based on pod labels, namespace selectors, or CIDR blocks. Requires a CNI that supports NetworkPolicy (Calico, Cilium — NOT Flannel).

### Q18: Explain the concept of taints and tolerations.
**A:** Taints are applied to nodes to repel pods. Tolerations are added to pods to allow scheduling on tainted nodes. Effects: `NoSchedule` (don't schedule), `PreferNoSchedule` (soft), `NoExecute` (evict existing). Used for: dedicated nodes, GPU nodes, control plane isolation.

### Q19: What is Helm and why use it?
**A:** Helm is a K8s package manager. Benefits: reusable templates (no copy-paste YAML), versioned releases with rollback, dependency management, environment-specific values files, community charts for common software. It simplifies deploying complex applications with many resources.

### Q20: How does Kubernetes handle secrets security?
**A:** By default, secrets are only base64 encoded (not encrypted). Best practices: enable encryption at rest (`EncryptionConfiguration`), use RBAC to restrict access, use external secret managers (Vault, AWS SM) with operators, avoid mounting secrets as env vars in logs, enable audit logging, rotate secrets regularly.

### Q21: What is the Container Storage Interface (CSI)?
**A:** CSI is a standard interface between K8s and storage providers. It allows storage vendors to develop plugins independently. The CSI driver handles volume provisioning, attaching, mounting, and snapshotting. Examples: EBS CSI, EFS CSI, Ceph CSI.

### Q22: How do you handle multi-cluster communication?
**A:** Options: Service mesh (Istio multi-cluster), cluster federation, DNS-based routing (ExternalName services), VPN/peering between cluster networks, or API gateway routing. Choice depends on latency requirements, security needs, and complexity tolerance.

---

## 16. Free Resources

| Resource | Link | Description |
|----------|------|-------------|
| Kubernetes Official Docs | https://kubernetes.io/docs/ | Comprehensive reference |
| Kubernetes the Hard Way | https://github.com/kelseyhightower/kubernetes-the-hard-way | Deep understanding of internals |
| KillerCoda | https://killercoda.com/ | Free interactive K8s labs |
| Play with Kubernetes | https://labs.play-with-k8s.com/ | Browser-based K8s playground |
| Kubernetes Podcast | https://kubernetespodcast.com/ | Weekly K8s news and interviews |
| CNCF Landscape | https://landscape.cncf.io/ | Ecosystem overview |
| Learnk8s | https://learnk8s.io/blog | Excellent articles and guides |
| kubectl Cheat Sheet | https://kubernetes.io/docs/reference/kubectl/cheatsheet/ | Quick reference |
| KodeKloud CKA Course | https://kodekloud.com/ | Affordable CKA prep with labs |
| Killer.sh | https://killer.sh/ | CKA exam simulator (2 free sessions with exam purchase) |
| Kubernetes Patterns Book | https://k8spatterns.io/ | Design patterns for K8s |
| Awesome Kubernetes | https://github.com/ramitsurana/awesome-kubernetes | Curated resource list |

---

## 17. CKA Exam Preparation Tips

### Exam Format
- **Duration:** 2 hours
- **Format:** Performance-based (hands-on CLI tasks in a real cluster)
- **Passing score:** 66%
- **Retake:** One free retake included
- **Resources allowed:** kubernetes.io/docs open in one tab

### Preparation Strategy

1. **Master `kubectl`** — the exam is all command-line, speed matters
2. **Practice imperative commands** — `kubectl run`, `kubectl create`, `kubectl expose` are faster than writing YAML
3. **Know `kubectl explain`** — use it instead of searching docs during exam
4. **Set up aliases and autocomplete:**
   ```bash
   alias k=kubectl
   source <(kubectl completion bash)
   complete -o default -F __start_kubectl k
   export do="--dry-run=client -o yaml"
   # Usage: k run nginx --image=nginx $do > pod.yaml
   ```
5. **Practice with time pressure** — do killer.sh simulator, aim to finish in 90 minutes
6. **Know these topics cold:** Deployments, Services, RBAC, NetworkPolicy, PV/PVC, Ingress, troubleshooting
7. **Practice `etcdctl` backup and restore** — almost guaranteed exam question
8. **Learn to upgrade clusters** — `kubeadm upgrade` workflow
9. **Practice multi-container pods** — sidecar, init containers
10. **Know jsonpath** — `kubectl get pods -o jsonpath='{.items[*].metadata.name}'`

### Time Management
- Read ALL questions first, do easy ones first (2-4 minutes each)
- Skip hard questions, return later (some are worth 4-8%)
- Don't waste 15 minutes on a 4% question
- Use bookmarks in the docs tab — pre-bookmark key pages

### Key Commands to Memorize

```bash
# Quick pod
k run nginx --image=nginx --port=80

# Quick deployment
k create deployment webapp --image=nginx --replicas=3

# Expose
k expose deployment webapp --port=80 --target-port=8080 --type=NodePort

# Quick service
k create service clusterip my-svc --tcp=80:8080

# ConfigMap
k create configmap my-config --from-literal=key=value

# Secret
k create secret generic my-secret --from-literal=pass=123

# ServiceAccount
k create serviceaccount my-sa

# Role
k create role pod-reader --verb=get,list --resource=pods

# RoleBinding
k create rolebinding read-pods --role=pod-reader --serviceaccount=default:my-sa

# Taint
k taint node node1 key=value:NoSchedule

# ETCD backup
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

## Summary Checklist

- [ ] Understand K8s architecture (control plane + worker nodes)
- [ ] Create and manage Pods, Deployments, Services
- [ ] Configure ConfigMaps and Secrets
- [ ] Set up Ingress with TLS
- [ ] Implement HPA and PDB for production readiness
- [ ] Configure RBAC with least privilege
- [ ] Write and apply NetworkPolicies
- [ ] Troubleshoot common pod/service issues
- [ ] Use Helm for application packaging
- [ ] Practice kubectl speed for CKA exam

---

> **Next:** Month 3 — CI/CD Pipelines, GitOps with ArgoCD, and Infrastructure as Code with Terraform
