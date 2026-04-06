# On-Demand-car-Wash-Docker-K8-s-Configuration-

# Docker, Kubernetes & CI/CD - Complete Guide

### Car Wash Microservices System

> A comprehensive guide covering Docker containerization, Kubernetes orchestration, and CI/CD pipeline implementation for a microservices-based on-demand car wash platform.

---

## Table of Contents

- [Part 1: Docker - Theory & Concepts](#part-1-docker---theory--concepts)
  - [What is Docker?](#11-what-is-docker)
  - [Key Docker Concepts](#12-key-docker-concepts)
    - [Docker Image](#121-docker-image)
    - [Docker Container](#122-docker-container)
    - [Dockerfile](#123-dockerfile)
    - [Multi-Stage Builds](#124-multi-stage-builds)
    - [.dockerignore](#125-dockerignore)
    - [Docker Volumes](#126-docker-volumes)
    - [Docker Networks](#127-docker-networks)
    - [Docker Compose](#128-docker-compose)
    - [Docker Health Checks](#129-docker-health-checks)
    - [Docker Hub](#1210-docker-hub-container-registry)
    - [Docker Security Best Practices](#1211-docker-security-best-practices)
- [Part 2: Kubernetes - Theory & Concepts](#part-2-kubernetes-k8s---theory--concepts)
  - [What is Kubernetes?](#21-what-is-kubernetes)
  - [Why Kubernetes?](#22-why-kubernetes-problems-it-solves)
  - [Key Kubernetes Concepts](#23-key-kubernetes-concepts)
    - [Cluster](#231-cluster)
    - [Node](#232-node)
    - [Pod](#233-pod)
    - [Deployment](#234-deployment)
    - [Service](#235-service)
    - [Namespace](#236-namespace)
    - [ConfigMap](#237-configmap)
    - [Secret](#238-secret)
    - [PersistentVolumeClaim](#239-persistentvolumeclaim-pvc)
    - [Probes](#2310-probes-health-checks-in-k8s)
    - [Resource Limits](#2311-resource-limits)
    - [Kind](#2312-kind-kubernetes-in-docker)
- [Part 3: CI/CD & GitHub Actions](#part-3-cicd--github-actions---theory--concepts)
  - [What is CI/CD?](#31-what-is-cicd)
  - [What is a Pipeline?](#32-what-is-a-cicd-pipeline)
  - [What is GitHub Actions?](#33-what-is-github-actions)
  - [GitHub Secrets](#34-github-secrets)
  - [Docker Hub PAT](#35-docker-hub-personal-access-token-pat)
  - [Smart Change Detection](#36-why-only-build-changed-services)
- [Part 4: Our Implementation](#part-4-our-car-wash-system-implementation)
  - [Architecture Overview](#41-architecture-overview)
  - [What Each Service Does](#42-what-each-service-does)
  - [Docker Compose Configuration](#43-docker-compose-configuration-explained)
  - [Kubernetes Deployment Structure](#44-kubernetes-deployment-structure)
  - [CI/CD Pipeline Structure](#45-cicd-pipeline-structure)
- [Part 5: Commands Reference](#part-5-every-command-we-used---explained)
  - [Docker Commands](#51-docker-commands)
  - [Kubernetes Commands](#52-kubernetes-commands)
  - [Git Commands](#53-git--github-commands)
- [Part 6: Files We Created](#part-6-files-we-created---line-by-line-explanation)
  - [Dockerfile Explained](#61-dockerfile-optimized)
  - [.dockerignore Explained](#62-dockerignore)
  - [docker-compose.yml Explained](#63-docker-composeyml-key-sections)
  - [Kubernetes YAML Explained](#64-kubernetes-yaml-files---key-sections)
  - [GitHub Actions Workflow Explained](#65-github-actions-workflow---line-by-line)
  - [Push Script Explained](#66-push-to-dockerhubsh-explained)
  - [Deploy Script Explained](#67-k8sdeploysh-explained)
- [Summary](#summary-what-we-achieved)

---

---

# Part 1: Docker - Theory & Concepts

---

## 1.1 What is Docker?

Docker is a platform that allows you to package your application along with all its dependencies (libraries, runtime, system tools, settings) into a single unit called a **"container."**

### Real World Analogy

> Imagine you cook a dish at home. It tastes perfect because you have the right stove, the right spices, the right temperature. Now someone asks you to cook the same dish at their house. Their stove is different, they don't have the same spices, and the result is different.
>
> **Docker is like a portable kitchen.** You pack your stove, your spices, your ingredients, everything into a shipping container. Now wherever that container goes, you can cook the exact same dish.

| Real World | Software |
|-----------|----------|
| The dish | Your application |
| The kitchen | The runtime environment |
| The shipping container | Docker container |

Before Docker, developers faced the classic problem:

> *"It works on my machine!"* -- Every developer ever

Docker solves this by ensuring that if it works in a Docker container on your machine, it will work the **EXACT same way** on any other machine running Docker.

---

## 1.2 Key Docker Concepts

### 1.2.1 Docker Image

A Docker image is a **READ-ONLY template** that contains:

- Your application code
- The runtime (e.g., Java 21, Node.js, Python)
- Libraries and dependencies
- System tools and settings
- Instructions on how to run the application

> **Think of an image like a recipe card.** It has all the instructions but it's not the actual food. You use the recipe (image) to cook the food (create a container).

An image is built from a file called a **"Dockerfile"** (explained below).

**Example - Our eureka-server image contains:**

| Component | Size |
|-----------|------|
| Alpine Linux base OS | ~5MB |
| Java 21 JRE runtime | ~120MB |
| Spring user creation | ~1KB |
| Our application jar | ~30MB |
| File permissions | ~1KB |
| **Total** | **~155MB** |

Images are **LAYERED**. Each instruction in a Dockerfile creates a new layer. Layers are cached, so if you change only your application code, Docker doesn't need to re-download Java - it reuses the cached layer.

---

### 1.2.2 Docker Container

A container is a **RUNNING INSTANCE** of an image. While an image is a template, a container is the actual running application.

```
Image     = Class in Java       (blueprint)
Container = Object in Java      (actual instance)
```

You can create **MULTIPLE containers** from the same image, just like you can create multiple objects from the same class.

**Key properties of containers:**

| Property | Description |
|----------|-------------|
| **Isolated** | Each container runs in its own isolated environment |
| **Lightweight** | Containers share the host OS kernel (unlike VMs) |
| **Portable** | Run the same container on any machine with Docker |
| **Ephemeral** | Containers can be created, stopped, and destroyed quickly |

---

### 1.2.3 Dockerfile

A Dockerfile is a text file with instructions to build a Docker image. Think of it as a **recipe that Docker follows step by step.**

| Instruction | Purpose |
|------------|---------|
| `FROM` | Start from a base image (like starting with a base OS) |
| `WORKDIR` | Set the working directory inside the container |
| `COPY` | Copy files from your computer into the container |
| `RUN` | Execute a command during build time |
| `EXPOSE` | Document which port the app uses (informational only) |
| `CMD` | Default command to run when container starts |
| `ENTRYPOINT` | Command that always runs when container starts |

**Difference between CMD and ENTRYPOINT:**

```dockerfile
# ENTRYPOINT: Cannot be overridden easily. Use for the main command.
ENTRYPOINT ["java", "-jar", "app.jar"]

# CMD: Can be overridden. Use for default arguments.
CMD ["--server.port=8080"]
```

---

### 1.2.4 Multi-Stage Builds

A multi-stage build uses **multiple FROM instructions** in one Dockerfile. This is important for keeping images small.

**Why do we need this?**

| Stage | Components | Size |
|-------|-----------|------|
| Build environment | JDK + Maven + Source + Dependencies | ~650MB |
| Runtime (what we actually need) | JRE + compiled .jar | ~150MB |

```
Without multi-stage: Your image = 650MB (includes build tools)
With multi-stage:    Your image = 150MB (only runtime)
```

> This is like cooking in a full kitchen (Stage 1) but then serving the food on a small plate (Stage 2). You don't carry the entire kitchen to the dining table.

```dockerfile
# Stage 1: BUILD (full kitchen)
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY . .
RUN ./mvnw clean package

# Stage 2: RUNTIME (small plate)
FROM eclipse-temurin:21-jre-alpine
COPY --from=build /app/target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### 1.2.5 .dockerignore

Just like `.gitignore` tells Git which files to ignore, `.dockerignore` tells Docker which files **NOT to copy** into the image.

**Why is this important?**

When Docker builds an image, it sends all files in the directory to the Docker daemon (called the "build context"). If you have large files like `.git` folder, IDE files, or compiled binaries, it slows down the build.

```
.git          # Git history (not needed in container)
.idea         # IntelliJ IDEA settings
*.iml         # IntelliJ module files
target/       # Compiled files (we rebuild inside Docker)
README.md     # Documentation (not needed at runtime)
```

---

### 1.2.6 Docker Volumes

Containers are **ephemeral** - when you delete a container, all data inside it is lost. Volumes solve this problem by storing data **OUTSIDE** the container.

> **Analogy:** A container is like a hotel room. When you check out, the room is cleaned and everything you left is thrown away. A volume is like a **safety deposit box** - your valuables persist even after you check out.

**In our project:**

| Volume | Purpose |
|--------|---------|
| `mysql-data` | Stores MySQL database files (users, orders, cars survive container restarts) |
| `grafana-data` | Stores Grafana dashboards and settings |

---

### 1.2.7 Docker Networks

By default, containers are isolated and **cannot communicate** with each other. Docker networks allow containers to talk to each other.

```
Our project uses: carwash-network (bridge network)

Without network:  user-service CANNOT reach mysql
With network:     user-service connects to mysql:3306
                  (Docker resolves "mysql" to the MySQL container's IP)
```

**Types of Docker Networks:**

| Type | Description | Our Use |
|------|-------------|---------|
| `bridge` | Containers on same host communicate | **We use this** |
| `host` | Container uses host's network directly | - |
| `overlay` | Containers on different machines communicate | Docker Swarm |
| `none` | No networking | - |

---

### 1.2.8 Docker Compose

Docker Compose is a tool for defining and running **MULTI-CONTAINER** applications. Instead of running 17 separate `docker run` commands, you define everything in one `docker-compose.yml` file.

**Without Docker Compose (17 manual commands):**

```bash
docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234567890 mysql:8.0
docker run -d --name rabbitmq -p 5672:5672 rabbitmq:3-management
docker run -d --name redis -p 6379:6379 redis
docker run -d --name eureka -p 8761:8761 eureka-server
# ... 13 more commands
```

**With Docker Compose (one command):**

```bash
docker compose up -d
```

**Key docker-compose.yml concepts:**

| Key | Purpose |
|-----|---------|
| `services` | Each container is defined as a "service" |
| `image` | Which Docker image to use |
| `build` | Path to Dockerfile to build the image |
| `ports` | Map container port to host port (`host:container`) |
| `environment` | Set environment variables |
| `volumes` | Mount volumes for persistent data |
| `depends_on` | Define startup order |
| `healthcheck` | Define how to check if the service is healthy |
| `networks` | Which network to connect to |

---

### 1.2.9 Docker Health Checks

A health check is a command that Docker runs periodically to check if a container is working properly.

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `test` | Command to run | `wget --spider -q http://localhost:8761/actuator/health` |
| `interval` | How often to check | `15s` |
| `timeout` | Max time for each check | `5s` |
| `retries` | Failures before "unhealthy" | `10` |
| `start_period` | Grace period before counting failures | `40s` |

```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "-q", "http://localhost:8761/actuator/health"]
  start_period: 40s    # Wait 40s before checking
  interval: 15s        # Check every 15 seconds
  timeout: 5s          # Each check has 5s to respond
  retries: 10          # 10 failures = unhealthy
```

> **Why `wget` instead of `curl`?** Our images use Alpine Linux, which has `wget` but NOT `curl`. Using `curl` in an Alpine image would always fail.

---

### 1.2.10 Docker Hub (Container Registry)

Docker Hub is like GitHub but for Docker images. It's a cloud service where you can store and share your Docker images.

```
GitHub     = stores your SOURCE CODE
Docker Hub = stores your DOCKER IMAGES
```

**Our Docker Hub account:** `sarthakpawar09`

| Image | URL |
|-------|-----|
| `sarthakpawar09/carwash-eureka-server:latest` | Eureka Server |
| `sarthakpawar09/carwash-config-server:latest` | Config Server |
| `sarthakpawar09/carwash-api-gateway:latest` | API Gateway |
| `sarthakpawar09/carwash-user-service:latest` | User Service |
| `sarthakpawar09/carwash-car-service:latest` | Car Service |
| `sarthakpawar09/carwash-order-service:latest` | Order Service |
| `sarthakpawar09/carwash-payment-service:latest` | Payment Service |
| `sarthakpawar09/carwash-rating-service:latest` | Rating Service |
| `sarthakpawar09/carwash-invoice-service:latest` | Invoice Service |
| `sarthakpawar09/carwash-admin-service:latest` | Admin Service |
| `sarthakpawar09/carwash-notification-service:latest` | Notification Service |

**Image Tags:**

| Tag | Meaning |
|-----|---------|
| `:latest` | Most recent version |
| `:v1.0` | Specific version |
| `:abc123` | Tagged with git commit hash |

**Why use Docker Hub?**

1. **Deploy anywhere** - Any server can pull your images and run them
2. **CI/CD** - GitHub Actions builds images and pushes to Docker Hub
3. **Sharing** - Others can pull and run your application
4. **Backup** - Your images are stored in the cloud

---

### 1.2.11 Docker Security Best Practices

**1. Non-Root User:**

By default, containers run as `root` (administrator). This is dangerous - if someone hacks into your container, they have full control.

```dockerfile
# Create a limited user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring
```

**2. JVM Container-Aware Flags:**

Java needs to know it's running inside a container with limited resources.

```dockerfile
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

| Flag | Purpose |
|------|---------|
| `-XX:+UseContainerSupport` | Read container memory limits |
| `-XX:MaxRAMPercentage=75.0` | Use max 75% of available RAM |

Without these flags, Java might try to use more memory than allowed and get killed (OOMKilled).

---

---

# Part 2: Kubernetes (K8s) - Theory & Concepts

---

## 2.1 What is Kubernetes?

Kubernetes (K8s) is a container **ORCHESTRATION** platform. If Docker is about running containers, Kubernetes is about **MANAGING** containers at scale.

```
Docker       = A single musician who can play an instrument
Kubernetes   = The conductor who manages an entire orchestra
```

```
Docker         tells you HOW to run a container.
Kubernetes     tells you WHERE, WHEN, and HOW MANY containers to run.
```

```
Docker Compose = Managing a small restaurant (1 location, 1 kitchen)
Kubernetes     = Managing a restaurant chain (multiple locations,
                 automatic scaling, self-healing)
```

---

## 2.2 Why Kubernetes? (Problems it Solves)

| Problem | Docker Compose | Kubernetes |
|---------|---------------|------------|
| **Scaling** | You manually decide how many containers to run | Automatically adds/removes containers based on load |
| **Self-Healing** | If a container crashes, you manually restart it | Automatically detects and restarts crashed containers |
| **Rolling Updates** | Stop old, start new (downtime) | Gradually replaces old with new (zero downtime) |
| **Service Discovery** | Container names | Built-in DNS + load balancing |
| **Resource Management** | No limits by default | CPU/memory limits per container |

**Example Scenarios:**

> **Scaling:** Black Friday sale -> traffic increases 10x -> K8s automatically creates more order-service containers
>
> **Self-Healing:** user-service crashes at 3 AM -> K8s detects it within seconds and starts a new container
>
> **Rolling Updates:** Deploy new api-gateway version -> K8s starts new version -> routes traffic -> stops old version -> zero downtime

---

## 2.3 Key Kubernetes Concepts

### 2.3.1 Cluster

A Kubernetes cluster is a set of machines (called **nodes**) that run containerized applications managed by Kubernetes.

**Our setup:** Kind (Kubernetes IN Docker)
- Runs a single-node K8s cluster inside Docker Desktop
- Great for development and testing
- Not suitable for production (single point of failure)

**Production clusters typically have:**
- 3+ master nodes (control plane)
- Multiple worker nodes (run your applications)

---

### 2.3.2 Node

A node is a single machine in the Kubernetes cluster.

| Node Type | Role |
|-----------|------|
| **Master Node (Control Plane)** | Manages the cluster, schedules pods, monitors health |
| **Worker Node** | Runs your application containers (pods), reports to master |

---

### 2.3.3 Pod

A pod is the **SMALLEST deployable unit** in Kubernetes. It wraps one or more containers.

```
Docker world:      Container is the smallest unit
Kubernetes world:  Pod is the smallest unit (contains 1+ containers)
```

Usually **1 pod = 1 container**, but sometimes pods have "sidecar" containers (e.g., a logging agent alongside your application).

**In our project:**

```
Pod: eureka-server-xxx   -> Container: eureka-server
Pod: user-service-xxx    -> Container: user-service
Pod: mysql-xxx           -> Container: mysql
```

---

### 2.3.4 Deployment

A Deployment manages pods. It ensures the desired number of pod replicas are running at all times.

**What a Deployment does:**

1. Creates pods based on a template
2. Maintains desired replica count (e.g., always 1 running)
3. Handles rolling updates (deploy new version without downtime)
4. Rolls back if a new version fails
5. Self-heals (if a pod crashes, creates a new one)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 1                                          # Always keep 1 pod running
  selector:
    matchLabels:
      app: user-service
  template:
    spec:
      containers:
        - name: user-service
          image: sarthakpawar09/carwash-user-service:latest
```

> If the user-service pod crashes, the Deployment controller detects it and creates a new pod automatically.

---

### 2.3.5 Service

A Kubernetes Service is a **stable network endpoint** for accessing pods.

**Why do we need Services?**

Pods are ephemeral - they can be created and destroyed at any time. Each time a pod is recreated, it gets a **NEW IP address**. So you can't rely on pod IP addresses.

A Service provides:
- A **stable IP address** (doesn't change even if pods restart)
- A **DNS name** (e.g., "mysql" resolves to the MySQL service)
- **Load balancing** (distributes traffic across multiple pods)

**Types of Services:**

| Type | Access | Example in Our Project |
|------|--------|----------------------|
| `ClusterIP` (default) | Internal only (within cluster) | mysql, rabbitmq, redis |
| `NodePort` | External via static port on each node | Grafana (nodePort: 30300) |
| `LoadBalancer` | External via cloud load balancer | API Gateway (entry point) |

---

### 2.3.6 Namespace

A namespace is a **virtual cluster** within a Kubernetes cluster. It provides isolation and organization.

> **Analogy:** Cluster = An office building. Namespace = A floor in the building. Each floor has its own rooms (pods), security (RBAC), and resources.

**Our namespace:** `carwash`

| Namespace | Purpose |
|-----------|---------|
| `default` | Where things go if you don't specify |
| `kube-system` | Kubernetes internal components |
| `carwash` | **Our application** |

---

### 2.3.7 ConfigMap

A ConfigMap stores **non-sensitive** configuration data as key-value pairs. Pods can read this data as environment variables or mounted files.

**In our project:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    scrape_configs:
      - job_name: 'user-service'
        metrics_path: '/actuator/prometheus'
        static_configs:
          - targets: ['user-service:8081']
```

This ConfigMap is mounted as a file inside the Prometheus pod at `/etc/prometheus/prometheus.yml`.

---

### 2.3.8 Secret

A Secret is like a ConfigMap but for **SENSITIVE data** (passwords, tokens). Secrets are base64-encoded.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: carwash-secrets
type: Opaque
stringData:
  mysql-root-password: "1234567890"
  mysql-username: "root"
```

**Referenced in Deployments:**

```yaml
env:
  - name: SPRING_DATASOURCE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: carwash-secrets
        key: mysql-root-password
```

> Instead of hardcoding passwords in YAML files, we reference the Secret. Much more secure.

---

### 2.3.9 PersistentVolumeClaim (PVC)

Like Docker volumes, PVCs provide **persistent storage** for pods.

```
Without PVC: Delete MySQL pod -> ALL DATA IS LOST
With PVC:    Delete MySQL pod -> Data preserved, new pod mounts same PVC
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi    # 1 Gigabyte of persistent storage
```

---

### 2.3.10 Probes (Health Checks in K8s)

Kubernetes uses **3 types of probes** to check container health:

| Probe | Purpose | On Failure |
|-------|---------|------------|
| **Startup Probe** | Has the app finished starting? | Keep waiting (don't kill) |
| **Readiness Probe** | Is the app ready for traffic? | Remove from load balancer (don't kill) |
| **Liveness Probe** | Is the app still alive? | **Kill and restart the pod** |

**Why we use Startup Probe:**

Our Spring Boot services take **60-180 seconds** to start. Without a startup probe, the liveness probe would kill the pod before it finishes starting.

```yaml
startupProbe:
  httpGet:
    path: /actuator/health
    port: 8081
  failureThreshold: 30     # Allow 30 failures
  periodSeconds: 10        # Check every 10 seconds
  # Total: 30 x 10 = 300 seconds (5 minutes) to start up
  
readinessProbe:
  httpGet:
    path: /actuator/health
    port: 8081
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /actuator/health
    port: 8081
  periodSeconds: 15
```

**Flow:**

```
App starting -> Startup Probe runs (up to 5 min)
                    |
                    v (Startup succeeds)
              Readiness + Liveness probes activate
                    |
              Readiness fails -> Remove from load balancer (no traffic)
              Liveness fails  -> Kill pod, create new one
```

---

### 2.3.11 Resource Limits

Kubernetes lets you define how much CPU and memory each container can use.

| Field | Meaning |
|-------|---------|
| `requests` | Minimum guaranteed resources |
| `limits` | Maximum allowed resources |

```yaml
resources:
  requests:
    memory: "256Mi"    # Guaranteed 256 MiB
    cpu: "100m"        # Guaranteed 0.1 CPU core
  limits:
    memory: "384Mi"    # Max 384 MiB
    cpu: "300m"        # Max 0.3 CPU core
```

**CPU units explained:**

| Value | Meaning |
|-------|---------|
| `1000m` | 1 full CPU core |
| `500m` | Half a CPU core |
| `100m` | 10% of a CPU core |

> **Without limits:** One runaway container can eat all server memory, crashing ALL containers.
> **With limits:** Each container is capped, one bad actor can't affect others.

---

### 2.3.12 Kind (Kubernetes IN Docker)

Kind creates a Kubernetes cluster using Docker containers as nodes. Instead of needing real servers, each "node" is a Docker container.

| Feature | Detail |
|---------|--------|
| **Runs on** | Your local machine (no cloud needed) |
| **Cost** | Free |
| **Best for** | Development and testing |
| **Limitation** | Single node = limited resources |

> Our 17 microservices need more memory than a single Kind node provides. That's why some services were crashing (OOM - Out Of Memory). For production, use a cloud-managed K8s (EKS, GKE, AKS).

---

---

# Part 3: CI/CD & GitHub Actions - Theory & Concepts

---

## 3.1 What is CI/CD?

| Term | Full Form | Meaning |
|------|-----------|---------|
| **CI** | Continuous Integration | Automatically build & test on every code push |
| **CD** | Continuous Delivery | Automatically prepare releases (deploy with button click) |
| **CD** | Continuous Deployment | Automatically deploy to production (no human intervention) |

### Analogy

> Imagine you're writing a book with 10 co-authors.
>
> **WITHOUT CI/CD:** Each author writes chapters separately for 6 months. At the end, you merge all chapters. Chapters conflict, styles don't match, references are broken. This is **"Integration Hell."**
>
> **WITH CI/CD:** Authors submit chapters daily. After each submission, an automated system checks spelling/grammar (CI), formats and adds the chapter to the book (CD). Everyone reads the latest version immediately.

---

## 3.2 What is a CI/CD Pipeline?

A pipeline is a series of **automated steps** that code goes through from commit to deployment.

```
  Developer pushes code to GitHub
           |
           v
  GitHub Actions triggers automatically
           |
           v
  Step 1: Detect which services changed
           |
           v
  Step 2: Build changed services with Maven
           |
           v
  Step 3: Build Docker images
           |
           v
  Step 4: Push images to Docker Hub
           |
           v
  Step 5: (Optional) Deploy to Kubernetes
```

> This entire process is **AUTOMATIC**. You just push code, and everything else happens without you doing anything.

---

## 3.3 What is GitHub Actions?

GitHub Actions is GitHub's built-in CI/CD platform. It allows you to automate workflows directly in your GitHub repository.

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| **Workflow** | Automated process in a YAML file (`.github/workflows/`) |
| **Job** | A unit of work within a workflow (runs on a fresh VM) |
| **Step** | A single task within a job (shell command or action) |
| **Action** | Reusable package (e.g., `actions/checkout@v4`) |
| **Runner** | Server that runs your workflow (GitHub provides free ones) |
| **Event/Trigger** | What causes a workflow to run (push, PR, schedule) |
| **Matrix** | Run same job with different configs (build 11 services in parallel) |

**Common Actions we used:**

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Checks out your code onto the runner |
| `actions/setup-java@v4` | Installs Java 21 |
| `actions/cache@v4` | Caches Maven dependencies between runs |
| `docker/login-action@v3` | Logs into Docker Hub |
| `docker/build-push-action@v5` | Builds and pushes Docker images |
| `dorny/paths-filter@v3` | Detects which files changed |

**Triggers:**

| Trigger | When it runs |
|---------|-------------|
| `push` | When code is pushed to a branch |
| `pull_request` | When a PR is opened or updated |
| `schedule` | On a cron schedule |
| `workflow_dispatch` | Manual trigger (button click) |

---

## 3.4 GitHub Secrets

GitHub Secrets are **encrypted environment variables** for sensitive data like API keys, passwords, and access tokens.

### Why We Need DOCKERHUB_TOKEN

Our CI/CD pipeline pushes Docker images to Docker Hub. To push, you need to be authenticated.

We **can't** put the Docker Hub password in the workflow file because it's in the git repository - anyone can see it!

**Instead:**

1. Store the Docker Hub token as a **GitHub Secret** (encrypted)
2. Reference it in the workflow as `${{ secrets.DOCKERHUB_TOKEN }}`
3. GitHub decrypts it **only when the workflow runs**
4. It's **never visible** in logs (GitHub masks it with `***`)

**How to create a GitHub Secret:**

1. Go to your GitHub repository
2. **Settings** -> **Secrets and variables** -> **Actions**
3. Click **"New repository secret"**
4. Name: `DOCKERHUB_TOKEN`
5. Value: Your Docker Hub Personal Access Token
6. Click **"Add secret"**

---

## 3.5 Docker Hub Personal Access Token (PAT)

A PAT is like a password but more secure:

| Feature | Password | PAT |
|---------|----------|-----|
| Permissions | Full access | Can be limited (read-only, read-write) |
| Revocation | Changes main password | Revoke without affecting main password |
| Multiple | Only one | Can have multiple for different purposes |

**Our PAT permissions:** Read & Write (pull + push images)

> **SECURITY:** NEVER share your PAT in chat, emails, or code files. If exposed, delete it immediately and create a new one.

---

## 3.6 Why Only Build Changed Services?

Our pipeline uses **path filters** to detect which services changed:

```yaml
filters:
  admin-service:
    - 'admin-service/**'    # If any file here changed
  api-gateway:
    - 'api-gateway/**'      # If any file here changed
```

| Scenario | Without Filters | With Filters |
|----------|----------------|--------------|
| Change 1 line in user-service | Rebuild ALL 11 services (22 min) | Rebuild only user-service (2 min) |
| Push 11 images | Yes (wasteful) | Push 1 image (efficient) |

---

---

# Part 4: Our Car Wash System Implementation

---

## 4.1 Architecture Overview

```
Client (Browser/App)
       |
       v
API Gateway (8080) -------- JWT Authentication + Rate Limiting
       |
       v
Eureka Server (8761) ------ Service Discovery
       |
       v
Config Server (8888) ------ Centralized Configuration (from GitHub)
       |
       v
+------------------+------------------+------------------+------------------+
|                  |                  |                  |                  |
User Service    Car Service    Order Service    Payment Service
(8081)          (8082)         (8083)           (8084)
|                  |                  |                  |
+------------------+------------------+------------------+------------------+
|                  |                  |                  |
Rating Service  Invoice Service  Admin Service    Notification Service
(8086)          (8087)           (8088)           (8089)

All services connect to:
  - MySQL (3306)      - Database
  - RabbitMQ (5672)   - Message Queue
  - Redis (6379)      - Caching (Gateway only)
  - Zipkin (9411)     - Distributed Tracing

Monitoring:
  - Prometheus (9090) - Collects metrics
  - Grafana (3001)    - Dashboards
```

---

## 4.2 What Each Service Does

### Infrastructure Services

| Service | Port | Purpose |
|---------|------|---------|
| **MySQL** | 3306 | Relational database storing all application data (users, cars, orders, ratings, invoices) |
| **RabbitMQ** | 5672 / 15672 | Message broker for async communication. Example: order placed -> notification sent via message queue |
| **Redis** | 6379 | In-memory cache for API Gateway rate limiting |
| **Zipkin** | 9411 | Distributed tracing - tracks requests across multiple services with timing |

### Discovery & Config

| Service | Port | Purpose |
|---------|------|---------|
| **Eureka Server** | 8761 | Service registry - all microservices register here. Enables dynamic discovery (no hardcoded URLs) |
| **Config Server** | 8888 | Centralized config management. All configs stored in GitHub repo, pushed via RabbitMQ on changes |

### Gateway

| Service | Port | Purpose |
|---------|------|---------|
| **API Gateway** | 8080 | Single entry point. JWT authentication, rate limiting (Redis), routes requests, forwards `X-User-Email` and `X-User-Role` headers |

### Business Services

| Service | Port | Purpose |
|---------|------|---------|
| **User Service** | 8081 | Registration, login, profile management, JWT token generation. Roles: CUSTOMER, WASHER, ADMIN |
| **Car Service** | 8082 | CRUD operations for car details, linked to customers |
| **Order Service** | 8083 | Create/manage wash orders, assign washers, track status |
| **Payment Service** | 8084 | Process payments, payment status tracking |
| **Rating Service** | 8086 | Customer ratings for completed washes |
| **Invoice Service** | 8087 | Generate invoices for completed orders. Circuit breaker (Resilience4j) |
| **Admin Service** | 8088 | Admin dashboard, manage users/orders/washers. Circuit breaker (Resilience4j) |
| **Notification Service** | 8089 | Email/SMS notifications via RabbitMQ messages |

### Monitoring

| Service | Port | Purpose |
|---------|------|---------|
| **Prometheus** | 9090 | Scrapes metrics from all services via `/actuator/prometheus` |
| **Grafana** | 3001 | Dashboards to visualize Prometheus metrics |

---

## 4.3 Docker Compose Configuration Explained

### Startup Order

Our `docker-compose.yml` defines 17 services with this dependency chain:

```
Level 0 (No dependencies - start first):
  mysql, rabbitmq, redis, zipkin, prometheus

Level 1 (Depends on Level 0):
  eureka-server (standalone, but others need it)
  grafana (depends on prometheus)

Level 2 (Depends on eureka + rabbitmq):
  config-server

Level 3 (Depends on config-server + various infra):
  api-gateway (+ redis)
  All 8 business services (+ mysql + rabbitmq)
```

The `depends_on` + `condition: service_healthy` ensures proper ordering:

```yaml
config-server:
  depends_on:
    eureka-server:
      condition: service_healthy    # Wait for Eureka health check to pass
    rabbitmq:
      condition: service_healthy    # Wait for RabbitMQ health check to pass
```

---

## 4.4 Kubernetes Deployment Structure

### File Organization

| Order | File | Contents |
|-------|------|----------|
| 1 | `namespace.yml` | Creates the `carwash` namespace |
| 2 | `secrets.yml` | Database credentials (encrypted) |
| 3 | `infrastructure.yml` | MySQL, RabbitMQ, Redis, Zipkin |
| 4 | `monitoring.yml` | Prometheus (with ConfigMap) + Grafana |
| 5 | `discovery-config.yml` | Eureka Server + Config Server |
| 6 | `gateway.yml` | API Gateway (LoadBalancer service) |
| 7 | `services.yml` | All 8 business services |

> **Deploy order matters:** `namespace -> secrets -> infrastructure -> monitoring -> discovery-config -> gateway -> services`

Each microservice deployment includes:
- Docker Hub image reference
- Environment variables
- Resource requests and limits
- Startup probe, readiness probe
- A ClusterIP Service for internal communication

---

## 4.5 CI/CD Pipeline Structure

### Workflow: `.github/workflows/build-and-push.yml`

**Trigger:** Push to `main` branch or Pull Request to `main`

```
JOB 1: detect-changes
  |  Uses dorny/paths-filter to check which directories changed
  |  Outputs: true/false for each of the 11 services
  |
  v
JOB 2: build-and-push (runs 11x in parallel via matrix)
  |  For each changed service:
  |    1. Checkout code
  |    2. Setup Java 21
  |    3. Cache Maven dependencies
  |    4. Build with Maven
  |    5. Login to Docker Hub (DOCKERHUB_TOKEN secret)
  |    6. Build & push Docker image (tags: latest + commit SHA)
  |
  v
JOB 3: deploy-k8s (optional, only on main push)
     Applies K8s manifests, restarts deployments
     Requires KUBE_CONFIG secret
```

---

---

# Part 5: Every Command We Used - Explained

---

## 5.1 Docker Commands

### `docker compose up -d`

```bash
docker    # Docker CLI tool
compose   # Subcommand for Docker Compose
up        # Create and start all services defined in docker-compose.yml
-d        # Detached mode (run in background, don't block terminal)
```

**What it does:**
1. Reads `docker-compose.yml`
2. Creates the network (`carwash-network`)
3. Pulls/builds images if needed
4. Creates and starts all 17 containers
5. Waits for health checks (`depends_on` conditions)

---

### `docker compose down`

```bash
docker compose down     # Stops and removes all containers + networks
docker compose down -v  # Also removes volumes (DATA LOSS!)
```

> Volumes are **NOT** removed by default (your data is preserved).

---

### `docker compose ps`

Lists all containers managed by Docker Compose. Shows: name, image, command, status, ports.

---

### `docker compose build`

```bash
docker compose build                    # Build all services (uses cache)
docker compose build --no-cache         # Rebuild everything from scratch
docker compose build admin-service      # Build only specific service
```

> Use `--no-cache` when you change a `pom.xml` or `Dockerfile` and cached layers cause issues.

---

### `docker login`

```bash
docker login -u sarthakpawar09
# Prompts for password (use PAT, not account password)
# Creates/updates: ~/.docker/config.json
```

---

### `docker tag`

```bash
docker tag carwashsystem-eureka-server:latest sarthakpawar09/carwash-eureka-server:latest
#          SOURCE_IMAGE:TAG                    TARGET_IMAGE:TAG
```

Creates a new tag (alias) for an existing image. Does NOT copy the image.

> Docker Hub requires image names to start with your username (`sarthakpawar09/`).

---

### `docker push`

```bash
docker push sarthakpawar09/carwash-eureka-server:latest
```

Uploads the image to Docker Hub. Only pushes layers that don't already exist.

| Log Message | Meaning |
|-------------|---------|
| `Layer already exists` | This layer is already on Docker Hub, skip |
| `Pushed` | This layer is new, uploading |
| `Mounted from ...` | Reusing a layer from another image in your account |

---

### `docker images`

Lists all images stored locally. Shows: repository, tag, image ID, size.

---

### `docker ps`

```bash
docker ps       # List running containers
docker ps -a    # List ALL containers (including stopped)
```

---

### `docker logs`

```bash
docker logs eureka-server --tail 50
#           CONTAINER_NAME    # Show last 50 lines
```

---

### `docker inspect`

```bash
docker inspect IMAGE --format='{{.Id}}'
# Shows detailed JSON info, --format extracts specific fields
```

---

### Windows Service Commands

```bash
net stop MySQL80    # Stop MySQL Windows service (frees port 3306)
net stop Grafana    # Stop Grafana Windows service (frees port 3000)
```

> Needed because Windows services conflict with Docker container ports.

---

## 5.2 Kubernetes Commands

### `kubectl apply`

```bash
kubectl apply -f namespace.yml
#       apply  = Create or update resources (DECLARATIVE)
#       -f     = From file
```

> `apply` is declarative: You declare the desired state, K8s figures out what to change. If the resource exists, it updates it. If not, it creates it.

---

### `kubectl get`

```bash
kubectl get pods -n carwash     # List all pods in carwash namespace
kubectl get svc -n carwash      # List all services
kubectl get nodes               # List all cluster nodes
```

---

### `kubectl describe`

```bash
kubectl describe pod POD_NAME -n carwash
```

Shows **detailed information**: container status, events, environment variables, resource usage. Very useful for debugging.

---

### `kubectl logs`

```bash
kubectl logs POD_NAME -n carwash --tail=30      # Last 30 lines
kubectl logs POD_NAME -n carwash --previous     # Logs from PREVIOUS crash
```

---

### `kubectl wait`

```bash
kubectl wait --for=condition=ready pod -l app=mysql -n carwash --timeout=300s
#            Wait for pods with label app=mysql to be Ready
#            Give up after 300 seconds (5 minutes)
```

---

### `kubectl rollout restart`

```bash
kubectl rollout restart deployment/admin-service -n carwash
```

Triggers a **rolling restart** - creates new pods, waits for ready, terminates old.

> Use when you pushed a new image with the same `:latest` tag and K8s needs to pull it.

---

### `kubectl delete`

```bash
kubectl delete namespace carwash
# DESTRUCTIVE: Deletes namespace AND everything inside it
```

---

### `kubectl scale`

```bash
kubectl scale rs REPLICASET_NAME --replicas=0 -n carwash
# Scale to 0 replicas (delete all pods in this ReplicaSet)
```

---

### `kubectl cluster-info`

```bash
kubectl cluster-info
# Shows K8s control plane address. Verifies cluster is running.
```

---

### `kubectl port-forward`

```bash
kubectl port-forward svc/grafana 3000:3000 -n carwash
# Forward local port 3000 -> Grafana service port 3000
# Access at http://localhost:3000
```

---

## 5.3 Git / GitHub Commands

```bash
git remote -v       # List all remote repositories
git status          # Show modified/untracked files
git push origin main # Push to remote (triggers GitHub Actions)
```

---

---

# Part 6: Files We Created - Line by Line Explanation

---

## 6.1 Dockerfile (Optimized)

```dockerfile
# ============================================
# Stage 1: Build
# ============================================
FROM eclipse-temurin:21-jdk-alpine AS build
```
- `FROM` - Start from this base image
- `eclipse-temurin:21-jdk-alpine` - Java 21 JDK on Alpine Linux (minimal ~5MB OS)
- `AS build` - Name this stage "build" (referenced later)

```dockerfile
WORKDIR /app
```
- Set working directory to `/app`. All subsequent commands run here.

```dockerfile
COPY .mvn .mvn
COPY mvnw pom.xml ./
```
- Copy Maven wrapper and `pom.xml` **FIRST** (before source code)
- **WHY?** Docker caches layers. If `pom.xml` doesn't change, the next step (dependency download) is cached. Saves **MINUTES** on rebuilds.

```dockerfile
RUN chmod +x mvnw && ./mvnw dependency:go-offline -B
```
- `chmod +x mvnw` - Make Maven wrapper executable
- `dependency:go-offline` - Download all dependencies
- `-B` - Batch mode (no interactive prompts)
- **This layer is cached** until `pom.xml` changes

```dockerfile
COPY src ./src
```
- Copy actual source code (changes frequently, so it's a separate layer)

```dockerfile
RUN ./mvnw clean package -Dmaven.test.skip=true -B
```
- `clean` - Delete previous build output
- `package` - Compile code and create `.jar` file
- `-Dmaven.test.skip=true` - Skip tests (faster build)

```dockerfile
# ============================================
# Stage 2: Runtime (only what's needed to RUN)
# ============================================
FROM eclipse-temurin:21-jre-alpine
```
- **NEW stage** from JRE (not JDK). JRE is smaller (~120MB vs ~400MB)
- The build stage is **discarded** - only the jar is kept

```dockerfile
RUN addgroup -S spring && adduser -S spring -G spring
```
- Create a non-root user named "spring"
- `-S` - System user (no home dir, no login shell)
- **Security:** App doesn't run as root

```dockerfile
WORKDIR /app
COPY --from=build /app/target/SERVICE-NAME.jar app.jar
```
- `--from=build` - Copy from Stage 1
- Copies **only** the compiled `.jar`, nothing else

```dockerfile
RUN chown spring:spring app.jar
USER spring
```
- Change file ownership to spring user
- Switch to spring user for all subsequent commands

```dockerfile
EXPOSE 8081
```
- **Informational only** - documents which port the app uses
- Does NOT actually open the port (you need `ports:` in docker-compose)

```dockerfile
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

| Part | Purpose |
|------|---------|
| `java` | Run Java |
| `-XX:+UseContainerSupport` | Tell JVM it's in a container |
| `-XX:MaxRAMPercentage=75.0` | Use max 75% of container's memory |
| `-jar app.jar` | Run the Spring Boot application |

---

## 6.2 .dockerignore

```
.git                        # Git history (can be 100MB+)
.gitignore                  # Not needed at runtime
.idea                       # IntelliJ IDEA project files
*.iml                       # IntelliJ module files
target/                     # Compiled output (rebuilt inside Docker)
!target/*.jar               # Exception: DO include .jar files
.mvn/wrapper/maven-wrapper.jar
README.md                   # Documentation
docker-compose*.yml         # Docker Compose files
k8s/                        # Kubernetes manifests
```

---

## 6.3 docker-compose.yml Key Sections

```yaml
eureka-server:
  build: ./eureka-server
  # Path to directory containing Dockerfile

  image: sarthakpawar09/carwash-eureka-server:latest
  # Name to tag the built image
  # When both build: and image: are present:
  #   docker compose build -> builds and tags with this name
  #   docker compose up -> uses this image

  ports:
    - "8761:8761"
  # HOST_PORT:CONTAINER_PORT
  # Access at http://localhost:8761

  environment:
    - EUREKA_CLIENT_REGISTER-WITH-EUREKA=false
  # Spring Boot reads these, overrides application.properties
  # "Don't register yourself with Eureka" (because this IS Eureka)

  healthcheck:
    test: ["CMD", "wget", "--spider", "-q", "http://localhost:8761/actuator/health"]
  # CMD     = Run as shell command
  # wget    = HTTP client (available in Alpine)
  # --spider = Don't download, just check accessibility
  # -q      = Quiet mode
  # /actuator/health = Spring Boot health endpoint -> {"status":"UP"}

  depends_on:
    config-server:
      condition: service_healthy
  # Don't start until config-server's health check passes
```

---

## 6.4 Kubernetes YAML Files - Key Sections

```yaml
apiVersion: apps/v1
# Kubernetes API version (apps/v1 for Deployments)

kind: Deployment
# Resource type: Deployment, Service, Secret, ConfigMap, etc.

metadata:
  name: user-service       # Unique name within namespace
  namespace: carwash       # Which namespace

spec:
  replicas: 1              # How many pod copies to run

  selector:
    matchLabels:
      app: user-service    # "Find pods with this label"

  template:
    metadata:
      labels:
        app: user-service  # Label must match selector above

    spec:
      containers:
        - name: user-service
          image: sarthakpawar09/carwash-user-service:latest
          # Pull from Docker Hub

          env:
            - name: SPRING_DATASOURCE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: carwash-secrets
                  key: mysql-root-password
          # Read value from Kubernetes Secret (not hardcoded)

          resources:
            requests:
              memory: "256Mi"    # Guaranteed minimum
              cpu: "100m"        # 0.1 CPU core
            limits:
              memory: "384Mi"    # Maximum allowed
              cpu: "300m"        # 0.3 CPU core
```

---

## 6.5 GitHub Actions Workflow - Line by Line

```yaml
name: Build & Push to Docker Hub
# Display name in GitHub UI
```

```yaml
on:
  push:
    branches: [main]         # Run on push to main
  pull_request:
    branches: [main]         # Run on PR to main
```

```yaml
env:
  DOCKER_HUB_USER: sarthakpawar09
  # Global variable for all jobs
```

```yaml
jobs:
  detect-changes:
    runs-on: ubuntu-latest   # GitHub-hosted runner (FREE)
    
    outputs:
      admin-service: ${{ steps.changes.outputs.admin-service }}
      # Make results available to other jobs

    steps:
      - uses: actions/checkout@v4
        # Check out repo code onto the runner

      - uses: dorny/paths-filter@v3
        id: changes
        with:
          filters: |
            admin-service:
              - 'admin-service/**'
        # If any file under admin-service/ changed -> output = true
```

```yaml
  build-and-push:
    needs: detect-changes    # Wait for detect-changes to finish
    
    strategy:
      matrix:
        service:
          - admin-service
          - api-gateway
          # ... 11 services
    # Matrix: Run this job 11 TIMES IN PARALLEL
```

```yaml
    steps:
      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ matrix.service }}-${{ hashFiles('...') }}
        # First run: slow (downloads deps). Next runs: fast (uses cache).
```

```yaml
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ env.DOCKER_HUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
        # Uses the GitHub Secret we added!
        # Only on push (PRs don't push images)
```

```yaml
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./${{ matrix.service }}     # Dockerfile location
          push: ${{ github.event_name == 'push' }}  # Push only on main
          tags: |
            sarthakpawar09/carwash-${{ matrix.service }}:latest
            sarthakpawar09/carwash-${{ matrix.service }}:${{ github.sha }}
          # Tag with :latest AND commit SHA
          cache-from: type=gha
          cache-to: type=gha,mode=max
          # Use GitHub Actions cache for Docker layers
```

---

## 6.6 push-to-dockerhub.sh Explained

```bash
#!/bin/bash
# Shebang - tells system to use bash

DOCKER_HUB_USER="sarthakpawar09"
# Docker Hub username

TAG="${1:-latest}"
# First script argument, defaults to "latest"
# Usage: ./push-to-dockerhub.sh v2.0  -> TAG=v2.0
#        ./push-to-dockerhub.sh       -> TAG=latest

SERVICES=( "admin-service" "api-gateway" ... )
# Bash array of all service names

for svc in "${SERVICES[@]}"; do
  # Loop through each service

  LOCAL_IMAGE="carwashsystem-${svc}:latest"
  # Image name from docker compose build

  REMOTE_IMAGE="${DOCKER_HUB_USER}/carwash-${svc}:${TAG}"
  # Docker Hub compatible name (username/repo:tag)

  docker tag "$LOCAL_IMAGE" "$REMOTE_IMAGE"
  # Create Docker Hub tag for local image

  docker push "$REMOTE_IMAGE"
  # Upload to Docker Hub
done
```

---

## 6.7 k8s/deploy.sh Explained

```bash
kubectl apply -f namespace.yml       # Create carwash namespace
kubectl apply -f secrets.yml         # Create database password secrets
kubectl apply -f infrastructure.yml  # Deploy MySQL, RabbitMQ, Redis, Zipkin

kubectl wait --for=condition=ready pod -l app=mysql -n carwash --timeout=120s
# PAUSE here until MySQL is running (max 120 seconds)

kubectl apply -f monitoring.yml       # Deploy Prometheus + Grafana
kubectl apply -f discovery-config.yml # Deploy Eureka + Config Server

kubectl wait --for=condition=ready pod -l app=eureka-server -n carwash --timeout=180s
# PAUSE until Eureka is ready

kubectl wait --for=condition=ready pod -l app=config-server -n carwash --timeout=180s
# PAUSE until Config Server is ready

kubectl apply -f gateway.yml          # Deploy API Gateway
kubectl apply -f services.yml         # Deploy all 8 business services
```

---

---

# Summary: What We Achieved

## 1. Docker Image Optimization

| Optimization | Benefit |
|-------------|---------|
| Multi-stage builds | Smaller images (~150MB vs ~650MB) |
| `.dockerignore` | Faster builds (smaller build context) |
| Non-root user | Security (limited privileges) |
| JVM container flags | Proper memory usage in containers |
| Maven layer caching | Faster rebuilds (skip dependency download) |

## 2. Docker Hub

| Item | Detail |
|------|--------|
| Account | `sarthakpawar09` |
| Images | 11 images at `sarthakpawar09/carwash-*:latest` |
| Push script | `push-to-dockerhub.sh` for manual pushes |

## 3. Kubernetes

| Item | Detail |
|------|--------|
| Manifests | 7 YAML files for complete deployment |
| Secrets | Database credentials in K8s Secrets |
| Resource limits | CPU + memory per container |
| Health probes | Startup + readiness probes |
| Monitoring | Prometheus + Grafana in K8s |
| Deploy script | `k8s/deploy.sh` with proper ordering |

## 4. CI/CD Pipeline

| Item | Detail |
|------|--------|
| Platform | GitHub Actions |
| Trigger | Push to `main` branch |
| Smart detection | Only rebuilds changed services |
| Parallelism | 11 services build simultaneously |
| Caching | Maven deps + Docker layers cached |
| Push | Automatic push to Docker Hub |
| Tags | `:latest` + git commit SHA |

## 5. Bug Fixes

| Fix | Details |
|-----|---------|
| 3 `pom.xml` files | Added missing `spring-boot-maven-plugin` (admin, notification, rating) |
| Health checks | Changed `curl` to `wget` for Alpine Linux |
| Docker Hub username | Corrected to `sarthakpawar09` |
| Startup timing | Added `start_period` for slow-starting services |

---

## Security Reminders

> 1. Delete the leaked Docker Hub PAT tokens
> 2. Never commit passwords or tokens to Git
> 3. Use GitHub Secrets for sensitive CI/CD data
> 4. Use Kubernetes Secrets for application passwords
> 5. Run containers as non-root users
> 6. Keep Docker images updated (security patches)

---

## Useful Links

| Resource | URL |
|----------|-----|
| Docker Documentation | https://docs.docker.com/ |
| Kubernetes Documentation | https://kubernetes.io/docs/ |
| GitHub Actions | https://docs.github.com/en/actions |
| Docker Hub | https://hub.docker.com/u/sarthakpawar09 |
| Spring Boot Actuator | https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html |
| Prometheus | https://prometheus.io/docs/ |
| Grafana | https://grafana.com/docs/ |

---

<p align="center">
  <b>Built with Spring Boot Microservices | Dockerized | Kubernetes Ready | CI/CD Automated</b>
</p>
