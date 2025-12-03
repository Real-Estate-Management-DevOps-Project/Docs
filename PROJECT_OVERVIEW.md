# Real Estate Management DevOps Project Overview

## Table of Contents
- [Introduction](#introduction)
- [Project Architecture](#project-architecture)
- [Application Components](#application-components)
- [DevOps Toolchain](#devops-toolchain)
- [Infrastructure Setup](#infrastructure-setup)
- [CI/CD Pipeline](#cicd-pipeline)
- [Deployment Strategy](#deployment-strategy)
- [Security Implementation](#security-implementation)
- [Getting Started](#getting-started)

---

## Introduction

This is a **production-ready, cloud-native real estate management platform** built with a microservices architecture and comprehensive DevOps practices. The project demonstrates enterprise-grade development, deployment, and operations using modern tools and GitOps methodology.

### Key Highlights
- 🏗️ **Microservices Architecture**: 5 backend services + 1 frontend
- 🐳 **Containerized**: Docker multi-stage builds for all services
- ☸️ **Kubernetes Native**: Full K8s deployment with Cilium CNI
- 🔄 **GitOps**: Automated deployments via ArgoCD
- 🔒 **Security First**: Network policies, mTLS, JWT authentication
- 🚀 **CI/CD**: Fully automated pipelines with GitHub Actions
- 📊 **Infrastructure as Code**: Ansible playbooks for cluster setup

---

## Project Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Users/Clients                         │
└────────────────────┬────────────────────────────────────┘
                     │ HTTPS (TLS)
                     ▼
        ┌────────────────────────────┐
        │   Cilium Gateway (L7)      │
        │   TLS: rs.nirmalsavinda.online
        └────────────┬───────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
    ▼                ▼                ▼
┌─────────┐    ┌──────────┐    ┌──────────┐
│ Client  │    │ API      │    │ Direct   │
│ React   │    │ Gateway  │    │ Services │
│Frontend │    │ (Go)     │    │          │
└────┬────┘    └────┬─────┘    └────┬─────┘
     │              │               │
     └──────────────┼───────────────┘
                    │
    ┌───────────────┼───────────────┬──────────────┐
    │               │               │              │
    ▼               ▼               ▼              ▼
┌─────────┐ ┌────────────┐ ┌──────────────┐ ┌──────────┐
│ Auth    │ │ User       │ │ Property     │ │ Lease    │
│ Service │ │ Service    │ │ Service      │ │ Service  │
│ (JWT)   │ │ (CRUD)     │ │ (CRUD)       │ │ (CRUD +  │
│         │ │            │ │              │ │  Cron)   │
└────┬────┘ └─────┬──────┘ └──────┬───────┘ └─────┬────┘
     │            │               │               │
     │            ▼               ▼               ▼
     │        ┌────────────────────────────────────┐
     │        │    PostgreSQL Databases (3)        │
     │        └────────────────────────────────────┘
     │
     ▼
┌─────────────────┐
│ Asgardeo/WSO2   │
│ Identity Server │
└─────────────────┘
```

### Directory Structure

```
Real-Estate-Management-DevOps-Project/
├── api-gateway/          # API Gateway service
│   ├── main.go
│   ├── Dockerfile
│   └── .github/workflows/
├── auth-service/         # JWT authentication service
│   ├── main.go
│   ├── Dockerfile
│   └── .github/workflows/
├── user-service/         # User management
│   ├── main.go
│   ├── Dockerfile
│   └── .github/workflows/
├── property-service/     # Property listings
│   ├── main.go
│   ├── Dockerfile
│   └── .github/workflows/
├── lease-service/        # Lease management
│   ├── main.go
│   ├── Dockerfile
│   └── .github/workflows/
├── client/               # React frontend
│   ├── src/
│   ├── Dockerfile
│   └── .github/workflows/
├── deployment/           # Infrastructure as Code
│   └── ansible/
│       ├── playbooks/
│       ├── roles/
│       └── group_vars/
└── argocd/              # GitOps repository
    └── apps/
        ├── user-service/
        ├── auth-service/
        ├── property-service/
        ├── lease-service/
        ├── api-gateway/
        ├── client/
        └── common/
```

---

## Application Components

### 1. **User Service** (Port 7202)
Handles user management and profiles.

**Features:**
- User registration and profile management
- Multiple user roles: buyer, seller, agent, developer, admin
- User search with pagination
- Profile ratings and experience tracking
- Saved searches functionality
- User activity tracking

**Tech Stack:**
- Go 1.24 + Fiber v2
- PostgreSQL 16 (GORM)
- Database: `real_estate_users`

### 2. **Auth Service** (Port 7201)
JWT token validation and OAuth2 integration.

**Features:**
- JWT token validation with JWKS
- OAuth2/OIDC integration (Asgardeo)
- Token issuer and audience verification
- Request authentication middleware

**Tech Stack:**
- Go 1.24 + Fiber v2
- JWT libraries (golang-jwt, MicahParks/keyfunc)
- No database (stateless)

### 3. **Property Service** (Port 7203)
Property listing and management.

**Features:**
- Property CRUD operations
- Property search with advanced filtering
- Multiple property types (residential, commercial, building)
- Media and document management
- Unit-level management within properties
- Location-based queries with coordinates
- Amenity tracking

**Tech Stack:**
- Go 1.24 + Fiber v2
- PostgreSQL 16 (GORM)
- Database: `real_estate_properties`

### 4. **Lease Service** (Port 7204)
Lease agreement and tenancy management.

**Features:**
- Lease agreement creation and management
- Digital signature workflow (landlord & tenant)
- Customizable lease templates
- Rent escalation with automatic increases
- Payment schedule generation and tracking
- Automated renewal reminders (90, 60, 30, 15, 7 days)
- Lease termination and eviction processes
- Security deposit tracking
- Late fee calculation
- Payment history and overdue detection
- Background cron jobs for automation

**Tech Stack:**
- Go 1.24 + Fiber v2
- PostgreSQL 16 (GORM)
- Database: `real_estate_leases`
- Cron job scheduler

### 5. **API Gateway** (Port 3000)
Routes requests to appropriate services.

**Features:**
- Request routing and forwarding
- Auth validation integration
- Service discovery
- Configuration-driven routing

**Tech Stack:**
- Go 1.24 + Fiber v2
- No database

### 6. **Client (Frontend)**
React-based web application.

**Features:**
- Modern React 19.2 with TypeScript
- OAuth2 authentication via Asgardeo
- Comprehensive UI component library
- Form management and validation
- Data visualization with charts
- Responsive design with TailwindCSS

**Tech Stack:**
- React 19.2.0 + TypeScript 5
- Vite build system
- TailwindCSS + Radix UI
- React Query for data fetching
- React Hook Form for forms
- Asgardeo auth integration

---

## DevOps Toolchain

### Containerization
**Docker** with multi-stage builds:
- Backend services use `scratch` or `alpine` base images
- CGO disabled for static binaries
- Binary optimization with `-ldflags='-s -w'`
- Minimal attack surface and image size

### Container Registry
**AWS ECR** (Elastic Container Registry):
- Account: `811329404514`
- Region: `us-east-1`
- 6 repositories (one per service)
- Images tagged with git SHA

### Orchestration
**Kubernetes 1.30.0**:
- Deployed via Ansible + kubeadm
- 2 replicas per service (high availability)
- Namespace: `real-estate`
- Resource management and limits
- Rolling update strategy

### Networking
**Cilium 1.18.3** (eBPF-based):
- Container Network Interface (CNI)
- Layer 7 load balancing
- Kubernetes Gateway API
- Network policies for micro-segmentation
- Hubble observability
- mTLS support
- Pod CIDR: `10.244.0.0/16`

### Certificate Management
**Cert-Manager + Let's Encrypt**:
- Automated TLS certificate provisioning
- ACME challenges over HTTP
- TLS termination at gateway
- Domain: `rs.nirmalsavinda.online`

### GitOps
**ArgoCD**:
- Declarative continuous deployment
- Separate Git repository for manifests
- ApplicationSet for app discovery
- Auto-sync with prune and self-heal
- Webhook-triggered updates

### Infrastructure as Code
**Ansible**:
- Automated Kubernetes cluster setup
- Modular roles for components
- Configuration management
- Idempotent playbooks

### CI/CD
**GitHub Actions**:
- Automated build and deployment
- AWS OIDC authentication (no static credentials)
- Secrets from AWS Secrets Manager
- Multi-stage pipelines
- Repository dispatch for GitOps

---

## Infrastructure Setup

### Ansible Roles

The project includes comprehensive Ansible playbooks for setting up the entire Kubernetes cluster:

#### 1. **Prerequisites Role**
- System updates (apt)
- Docker installation
- Kernel modules (br_netfilter, overlay)
- Network configuration (IP forwarding, bridge networking)
- Time synchronization

#### 2. **Kube Components Role**
- Kubernetes apt repository setup
- Install kubeadm, kubelet, kubectl (v1.30.0)
- Hold packages to prevent auto-updates

#### 3. **Control Plane Role**
- Initialize cluster with `kubeadm init`
- Configure pod CIDR: `10.244.0.0/16`
- Set up kubeconfig for admin access
- Generate join token for workers

#### 4. **Workers Role**
- Join worker nodes to cluster
- Configure kubelet

#### 5. **Cilium Role**
- Install Cilium CLI
- Deploy Cilium CNI (v1.18.3)
- Enable Gateway API support
- Configure Hubble observability

#### 6. **Tools Role**
- Install Helm package manager
- Deploy ArgoCD
- Install Kubernetes Dashboard
- Set up cert-manager

#### 7. **Apps Role**
- Deploy applications via ArgoCD ApplicationSet
- Configure Gateway and HTTPRoutes
- Set up TLS certificates

#### 8. **ECR Auth Role**
- Configure ECR authentication
- Create Kubernetes secret for image pull
- AWS credential management

### Configuration Variables

Key settings in `deployment/ansible/group_vars/all.yml`:

```yaml
k8s_version: "1.30.0"
pod_cidr: "10.244.0.0/16"
vpc_cidr: "10.0.0.0/8"
cilium_version: "1.18.3"
ecr_account_id: "811329404514"
ecr_region: "us-east-1"
```

---

## CI/CD Pipeline

### Pipeline Architecture

Each service has an automated CI/CD pipeline:

```
┌─────────────────────┐
│  Code Push (main)   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ GitHub Actions      │
│ (.github/workflows) │
└──────────┬──────────┘
           │
           ├──► Checkout code
           ├──► AWS OIDC auth
           ├──► Fetch secrets (AWS Secrets Manager)
           ├──► Docker build (multi-stage)
           ├──► ECR login
           ├──► Push image (tagged with git SHA)
           └──► Trigger GitOps workflow
                │
                ▼
┌─────────────────────────────┐
│  ArgoCD Repository          │
│  (repository_dispatch)      │
└──────────┬──────────────────┘
           │
           ├──► Update deployment YAML
           ├──► Update image URI with new tag
           ├──► Create PR
           ├──► Auto-merge to main
           └──► Commit changes
                │
                ▼
┌─────────────────────────────┐
│  ArgoCD Detects Changes     │
└──────────┬──────────────────┘
           │
           ├──► ApplicationSet discovers apps
           ├──► Sync deployment
           ├──► Rolling update pods
           ├──► Auto-heal on drift
           └──► Prune old resources
                │
                ▼
┌─────────────────────────────┐
│  Application Running        │
│  (New version deployed)     │
└─────────────────────────────┘
```

### Pipeline Features

**Security:**
- AWS OIDC federation (no static credentials)
- Secrets fetched from AWS Secrets Manager
- Image scanning (optional)
- Dependency vulnerability scanning via Dependabot

**Efficiency:**
- Multi-stage Docker builds
- Layer caching
- Only triggers on main branch push
- Parallel builds possible

**Automation:**
- Fully automated from code to production
- No manual intervention required
- Automatic PR creation and merge
- Self-healing deployments

### Environment Variables

Backend services use environment variables for configuration:

```bash
# Database
DB_HOST=postgres-service
DB_PORT=5432
DB_USER=admin
DB_PASSWORD=<from-k8s-secret>
DB_NAME=real_estate_<service>

# Server
PORT=3000
GIN_MODE=release

# Auth Service specific
JWKS_URL=https://api.asgardeo.io/t/<tenant>/oauth2/jwks
TOKEN_ISSUER=https://api.asgardeo.io/t/<tenant>/oauth2/token
AUDIENCE=<client-id>
```

Frontend uses `VITE_` prefixed variables:

```bash
VITE_API_URL=https://rs.nirmalsavinda.online
VITE_AUTH_CLIENT_ID=<asgardeo-client-id>
VITE_AUTH_BASE_URL=https://api.asgardeo.io/t/<tenant>
```

---

## Deployment Strategy

### Kubernetes Resources

Each service deployment includes:

#### 1. **Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <service>-deployment
  namespace: real-estate
spec:
  replicas: 2  # High availability
  selector:
    matchLabels:
      app: <service>
  template:
    spec:
      containers:
      - name: <service>
        image: 811329404514.dkr.ecr.us-east-1.amazonaws.com/real-estate-<service>:<tag>
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          value: postgres-<service>
        # ... more env vars
```

#### 2. **Service** (ClusterIP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: <service>-service
  namespace: real-estate
spec:
  type: ClusterIP
  ports:
  - port: 3000
    targetPort: 3000
  selector:
    app: <service>
```

#### 3. **PostgreSQL Database**
For services requiring persistence:
- Deployment with PostgreSQL 16
- PersistentVolumeClaim (10Gi)
- Secret for credentials
- Service for database access

#### 4. **NetworkPolicy** (Cilium)
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: <service>-netpol
spec:
  endpointSelector:
    matchLabels:
      app: <service>
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: api-gateway
    toPorts:
    - ports:
      - port: "3000"
        protocol: TCP
```

#### 5. **PeerAuthentication** (mTLS)
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: <service>-mtls
spec:
  selector:
    matchLabels:
      app: <service>
  mtls:
    mode: STRICT
```

### Gateway Configuration

**GatewayClass:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: cilium
spec:
  controllerName: io.cilium/gateway-controller
```

**Gateway:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: real-estate-gateway
spec:
  gatewayClassName: cilium
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: rs.nirmalsavinda.online
    tls:
      certificateRefs:
      - name: real-estate-tls
```

**HTTPRoutes:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-gateway-route
spec:
  parentRefs:
  - name: real-estate-gateway
  hostnames:
  - rs.nirmalsavinda.online
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-gateway-service
      port: 3000
```

### Certificate Management

**ClusterIssuer:**
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    - http01:
        gatewayHTTPRoute:
          parentRefs:
          - name: real-estate-gateway
```

**Certificate:**
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: real-estate-tls
spec:
  secretName: real-estate-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - rs.nirmalsavinda.online
```

---

## Security Implementation

### 1. **Authentication & Authorization**

**JWT-based authentication:**
- OAuth2/OIDC integration with Asgardeo (WSO2 Identity Server)
- JWT tokens validated by auth-service
- JWKS-based signature verification
- Token passed via Authorization header
- Token forwarded to backend services

**Flow:**
```
Client → Login (Asgardeo) → JWT Token
  ↓
Client Request + JWT → API Gateway
  ↓
API Gateway → Auth Service (validate token)
  ↓
Auth Service → JWKS validation → ✓ Valid
  ↓
API Gateway → Backend Service (with validated token)
  ↓
Backend Service → Process request
```

### 2. **Network Security**

**Cilium NetworkPolicies:**
- Default deny all traffic
- Explicit allow rules for service-to-service communication
- Port-level restrictions (only port 3000 allowed)
- Label-based endpoint selection
- Zero-trust networking model

**Example policy:**
```yaml
# Only API Gateway can talk to User Service on port 3000
ingress:
- fromEndpoints:
  - matchLabels:
      app: api-gateway
  toPorts:
  - ports:
    - port: "3000"
      protocol: TCP
```

### 3. **Transport Security**

**TLS/SSL:**
- Let's Encrypt certificates via cert-manager
- TLS termination at Cilium Gateway
- HTTPS enforcement on all external traffic
- Certificate auto-renewal

**mTLS (Service Mesh):**
- Mutual TLS between services
- PeerAuthentication in STRICT mode
- Encrypted service-to-service communication
- Identity verification

### 4. **Secrets Management**

**AWS Secrets Manager:**
- Store sensitive configuration
- Fetched during CI/CD build time
- No secrets in Git repositories
- Automatic rotation support

**Kubernetes Secrets:**
- Database credentials stored as K8s Secrets
- Mounted as environment variables
- Namespace-scoped access
- Base64 encoded

### 5. **Container Security**

**Image security:**
- Multi-stage builds (minimal final image)
- Scratch or alpine base images
- No unnecessary tools or packages
- Static binaries (no dynamic dependencies)
- Non-root user (where applicable)

**Registry security:**
- Private ECR repositories
- Image pull secrets in Kubernetes
- IAM-based access control
- Image scanning for vulnerabilities

### 6. **CI/CD Security**

**GitHub Actions:**
- AWS OIDC federation (no long-lived credentials)
- Short-lived tokens
- Secrets stored in AWS Secrets Manager
- Least privilege IAM roles
- Audit logging

**Supply chain security:**
- Dependabot for dependency updates
- Vulnerability scanning
- Automated security patches
- Pinned versions in Dockerfiles

---

## Getting Started

### Prerequisites

- Kubernetes cluster (1.30.0+)
- Ansible for cluster setup
- Docker for local development
- AWS account with ECR access
- Domain name for public access
- GitHub account for CI/CD

### Local Development

#### Backend Services

Each service supports local development:

```bash
# Navigate to service directory
cd user-service

# Start local PostgreSQL
docker-compose up -d

# Create .env file
cat > .env <<EOF
DB_HOST=localhost
DB_PORT=5432
DB_USER=admin
DB_PASSWORD=password
DB_NAME=real_estate_users
PORT=7202
EOF

# Install air for live reload
go install github.com/cosmtrek/air@latest

# Run with live reload
air

# Or build and run manually
go build -o main main.go
./main
```

#### Frontend

```bash
cd client

# Install pnpm globally
npm install -g pnpm

# Install dependencies
pnpm install --frozen-lockfile

# Create .env file
cat > .env <<EOF
VITE_API_URL=http://localhost:3000
VITE_AUTH_CLIENT_ID=your-client-id
VITE_AUTH_BASE_URL=https://api.asgardeo.io/t/your-tenant
EOF

# Run development server
pnpm dev

# Build for production
pnpm build
```

### Cluster Deployment

#### 1. Set up Kubernetes Cluster

```bash
cd deployment/ansible

# Update inventory with your nodes
vim inventory/hosts

# Update configuration
vim group_vars/all.yml

# Run playbooks in order
ansible-playbook -i inventory/hosts playbooks/01-prerequisites.yml
ansible-playbook -i inventory/hosts playbooks/02-kube-components.yml
ansible-playbook -i inventory/hosts playbooks/03-control-plane.yml
ansible-playbook -i inventory/hosts playbooks/04-workers.yml
ansible-playbook -i inventory/hosts playbooks/05-cilium.yml
ansible-playbook -i inventory/hosts playbooks/06-tools.yml
ansible-playbook -i inventory/hosts playbooks/07-ecr-auth.yml
ansible-playbook -i inventory/hosts playbooks/08-apps.yml
```

#### 2. Configure CI/CD

**For each service repository:**

1. Add GitHub secrets:
   ```
   AWS_ROLE_ARN: arn:aws:iam::811329404514:role/github-actions-ecr
   AWS_REGION: us-east-1
   ECR_REPOSITORY: real-estate-<service>
   ARGOCD_REPO: <your-username>/argocd
   GITHUB_APP_TOKEN: <your-github-app-token>
   ```

2. Push to main branch to trigger build

**For ArgoCD repository:**

1. Add GitHub secrets:
   ```
   GH_TOKEN: <github-personal-access-token>
   ```

2. Configure ArgoCD ApplicationSet:
   ```bash
   kubectl apply -f argocd/applicationset.yaml
   ```

#### 3. Access Application

```bash
# Get gateway external IP
kubectl get gateway real-estate-gateway -n real-estate

# Update DNS records
# A record: rs.nirmalsavinda.online → <gateway-ip>

# Wait for certificate to be issued
kubectl get certificate -n real-estate

# Access application
# https://rs.nirmalsavinda.online
```

### Monitoring and Troubleshooting

```bash
# Check pod status
kubectl get pods -n real-estate

# View logs
kubectl logs -f deployment/<service>-deployment -n real-estate

# Check ArgoCD sync status
kubectl get applications -n argocd

# View Cilium status
cilium status

# Check network policies
kubectl get ciliumnetworkpolicies -n real-estate

# View gateway configuration
kubectl describe gateway real-estate-gateway -n real-estate

# Check certificates
kubectl describe certificate real-estate-tls -n real-estate
```

---

## Summary

This project demonstrates a **complete DevOps lifecycle** for a microservices-based application:

- ✅ **Modern Architecture**: Cloud-native microservices with proper separation of concerns
- ✅ **Containerization**: Optimized Docker images with multi-stage builds
- ✅ **Orchestration**: Kubernetes with advanced networking (Cilium)
- ✅ **GitOps**: Automated deployments via ArgoCD
- ✅ **CI/CD**: Fully automated pipelines from code to production
- ✅ **Security**: Network policies, mTLS, JWT auth, TLS encryption
- ✅ **Infrastructure as Code**: Ansible playbooks for reproducible infrastructure
- ✅ **Observability**: Logging, monitoring, and tracing capabilities
- ✅ **High Availability**: Multiple replicas and rolling updates
- ✅ **Scalability**: Horizontal scaling and load balancing

This is an excellent reference project for learning and implementing production-grade DevOps practices in a real-world scenario.
