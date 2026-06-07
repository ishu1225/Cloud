# JENKINS + PROMETHEUS + GRAFANA — Viva + Exam Notes

---

## JENKINS — CORE CONCEPTS

|Term|Meaning|
|---|---|
|Pipeline|Automated sequence of steps (build, test, deploy)|
|Jenkinsfile|File that defines the pipeline as code|
|Stage|A major phase in pipeline (Build, Push, Deploy)|
|Step|Single action inside a stage|
|Agent|Machine that runs the pipeline|
|Credentials|Stored secrets (Docker Hub login, tokens)|
|Webhook|GitHub notifies Jenkins on every git push|
|Build Number|Auto-incrementing number for each pipeline run|

---

## HOW JENKINS WORKS IN THIS LAB

```
git push → GitHub webhook → Jenkins triggers
    ↓
Stage 1: Checkout code from GitHub
Stage 2: docker build → creates image with build number tag
Stage 3: docker push → sends image to Docker Hub
Stage 4: kubectl apply → deploys to Minikube
    ↓
App live on Minikube with new version
```

---

## JENKINSFILE — WRITE FROM SCRATCH IN EXAM

```groovy
pipeline {
    agent any

    environment {
        IMAGE = "zen1tsu/devops-app"
        TAG   = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                bat "docker build -t %IMAGE%:%TAG% ."
                bat "docker tag %IMAGE%:%TAG% %IMAGE%:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin"
                    bat "docker push %IMAGE%:%TAG%"
                    bat "docker push %IMAGE%:latest"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                bat "copy C:\\Users\\himan\\.kube\\config C:\\Windows\\System32\\config\\systemprofile\\.kube\\config /Y"
                bat "kubectl apply -f k8s\\deployment.yml --validate=false"
                bat "kubectl apply -f k8s\\service.yml --validate=false"
                bat "kubectl set image deployment/devops-deploy devops-app=%IMAGE%:%TAG%"
                bat "kubectl rollout status deployment/devops-deploy"
            }
        }

    }

    post {
        always {
            bat "docker rmi %IMAGE%:%TAG% || true"
        }
    }
}
```

**Important notes for exam:**

- Windows Jenkins uses `bat` not `sh`
- `${BUILD_NUMBER}` auto-increments each run
- `withCredentials` keeps Docker Hub password out of logs
- `post { always }` cleans up local image after push
- `--validate=false` avoids kubeconfig validation errors
- kubeconfig copy step is needed because Jenkins runs as System user

---

## JENKINS SETUP STEPS (if asked in exam)

```
1. Install Java 17 (Jenkins requires it)
2. Download Jenkins .msi from jenkins.io
3. Install → point to Java 17 path
4. Open http://localhost:8080
5. Get password: type C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword
6. Install suggested plugins
7. Create admin user
8. Manage Jenkins → Credentials → Add dockerhub-creds
9. New Item → Pipeline → SCM → Git → your repo URL
10. Add Jenkinsfile to repo root
11. GitHub → repo Settings → Webhooks → add Jenkins URL/github-webhook/
```

---

## SCENARIO — Full pipeline demo for exam

```powershell
# 1. Make a change to app.py
# Change return message to new version

# 2. Push to GitHub
git add .
git commit -m "update app v2"
git push origin main

# 3. Webhook triggers Jenkins automatically
# Watch at http://localhost:8080

# 4. After pipeline completes verify:
kubectl get pods                          # new pods with updated image
kubectl rollout history deployment/devops-deploy   # shows versions
minikube service devops-service --url     # get URL
curl http://127.0.0.1:<PORT>              # shows new version response
```

---

## SCENARIO — Manual pipeline trigger

```
Jenkins UI → devops-pipeline → Build Now
```

Or trigger via API:

```powershell
Invoke-WebRequest -Uri "http://localhost:8080/job/devops-pipeline/build" `
  -Method POST `
  -Credential (Get-Credential)
```

---

## JENKINS TROUBLESHOOTING

|Error|Cause|Fix|
|---|---|---|
|dial tcp refused|Minikube stopped or kubeconfig stale|`minikube start` then copy kubeconfig|
|error validating|kubectl can't reach cluster API|Add `--validate=false`|
|Cannot find path|USERNAME has special char like $|Hardcode full path in bat command|
|Docker not found|Docker not in Jenkins PATH|Add Docker bin to system PATH|
|Authentication required|Wrong credentials ID|Check credentialsId matches exactly|

---

## PROMETHEUS — CORE CONCEPTS

|Term|Meaning|
|---|---|
|Scrape|Prometheus pulls metrics from a target endpoint|
|Target|Service exposing metrics (Docker, app, node)|
|Job|Group of targets with same purpose|
|Metric|A single measurable value (CPU %, memory bytes)|
|PromQL|Query language to filter and aggregate metrics|
|Exporter|Agent that exposes metrics in Prometheus format|

---

## HOW PROMETHEUS WORKS

```
Docker engine  ──exposes──▶  :9323/metrics  ◀──scrapes──  Prometheus
cAdvisor       ──exposes──▶  :8080/metrics  ◀──scrapes──  Prometheus
Node Exporter  ──exposes──▶  :9100/metrics  ◀──scrapes──  Prometheus
                                                               ↓
                                                           Grafana queries
```

---

## prometheus.yml — WRITE FROM SCRATCH IN EXAM

```yaml
global:
  scrape_interval: 15s          # how often to scrape

scrape_configs:

  - job_name: 'prometheus'      # scrape itself
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'docker-engine'   # Docker daemon metrics
    static_configs:
      - targets: ['host.docker.internal:9323']

  - job_name: 'cadvisor'        # container metrics
    static_configs:
      - targets: ['devops-cadvisor:8080']

  - job_name: 'node-exporter'   # host machine metrics
    static_configs:
      - targets: ['devops-node-exporter:9100']
```

---

## PROMETHEUS COMMANDS FOR EXAM

```powershell
# Check if Prometheus is healthy
curl http://localhost:9090/-/healthy

# Reload config after editing prometheus.yml
Invoke-WebRequest -Uri "http://localhost:9090/-/reload" -Method POST

# Check targets via API
curl http://localhost:9090/api/v1/targets

# Test if container can reach Docker metrics
docker exec devops-prometheus wget -qO- http://host.docker.internal:9323/metrics
```

---

## USEFUL PROMQL QUERIES (show in exam)

```
# Running containers count
engine_daemon_container_states_containers{state="running"}

# CPU usage per container
rate(container_cpu_usage_seconds_total[1m])

# Memory usage per container
container_memory_usage_bytes

# Network bytes received
rate(container_network_receive_bytes_total[1m])

# Network bytes sent
rate(container_network_transmit_bytes_total[1m])
```

Steps to run query:

1. Open `http://localhost:9090/graph`
2. Type query in search box
3. Click Execute
4. Switch to Graph tab to see live chart

---

## GRAFANA — CORE CONCEPTS

|Term|Meaning|
|---|---|
|Data Source|Where Grafana reads metrics from (Prometheus)|
|Dashboard|Collection of panels showing graphs|
|Panel|Single graph or visualization|
|Import|Load a pre-built dashboard using ID from grafana.com|

---

## GRAFANA SETUP STEPS (exam)

```
1. Open http://localhost:3000
2. Login: admin / admin123
3. Left sidebar → Connections → Data Sources
4. Add data source → Prometheus
5. URL: http://devops-prometheus:9090
6. Save & Test → must show "Successfully queried"
7. Dashboards → Import → enter ID → Load → select datasource → Import
```

Dashboard IDs to know:

```
11600  → cAdvisor container metrics (CPU, memory, network per container)
1860   → Node Exporter host metrics (system-level)
193    → Docker container monitoring
```

---

## GENERATE TRAFFIC TO SHOW GRAPHS

```powershell
# Hit your app repeatedly so graphs show spikes
for ($i=0; $i -lt 50; $i++) {
    Invoke-WebRequest -Uri "http://localhost:9082" | Out-Null
}
```

Refresh Grafana dashboard after running this.

---

## DOCKER COMPOSE — START/STOP LAB

```powershell
# Start everything
docker compose up -d

# Stop everything
docker compose stop

# Restart specific service
docker compose restart devops-prometheus

# View logs of a service
docker compose logs devops-prometheus
docker compose logs devops-grafana

# Check all service status
docker compose ps
```

---

## FULL LAB STARTUP SEQUENCE (exam day)

```powershell
# 1. Start Docker Compose services
cd E:\devops-lab
docker compose up -d

# 2. Start Minikube
minikube start --driver=docker

# 3. Verify all running
docker compose ps
kubectl get nodes
kubectl get pods

# 4. Jenkins is already running as Windows Service
# Verify: Get-Service -Name Jenkins

# 5. Check all URLs
# http://localhost:8080   → Jenkins
# http://localhost:9082   → Flask app
# http://localhost:9090   → Prometheus
# http://localhost:3000   → Grafana
```

---

## VIVA QUESTIONS

**Q: What is CI/CD?** A: CI (Continuous Integration) automatically builds and tests code on every commit. CD (Continuous Delivery/Deployment) automatically deploys tested code to production. Jenkins automates this entire pipeline.

**Q: What is a Jenkinsfile?** A: A text file checked into the Git repo that defines the pipeline as code. It allows versioning the pipeline itself alongside the application code.

**Q: Why do we use webhooks?** A: Webhooks allow GitHub to notify Jenkins immediately when code is pushed, triggering the pipeline automatically without polling.

**Q: What is the difference between Prometheus and Grafana?** A: Prometheus collects and stores metrics by scraping endpoints. Grafana is a visualization tool that queries Prometheus and displays data as graphs and dashboards.

**Q: What is an exporter in Prometheus?** A: An exporter is an agent that exposes metrics in a format Prometheus can scrape. Examples: node-exporter for system metrics, cAdvisor for container metrics.

**Q: What is scrape interval?** A: How frequently Prometheus collects metrics from each target. Default is 15 seconds in our config.

**Q: Why does Jenkins need kubeconfig?** A: kubectl needs kubeconfig to know the cluster API address and credentials. Jenkins runs as a system user that doesn't have access to the user's kubeconfig by default, so we copy it manually.

**Q: What is Docker-in-Docker and why did we avoid it?** A: Running Docker inside a Docker container (Jenkins container trying to build Docker images). It causes socket permission issues and security problems. We run Jenkins natively on Windows so it uses Docker Desktop directly.

---

## EXAM TIP — If asked to show monitoring

```powershell
# 1. Show Prometheus targets UP
# Open: http://localhost:9090/targets

# 2. Run a query
# Open: http://localhost:9090/graph
# Query: engine_daemon_container_states_containers{state="running"}
# Click Execute → Graph tab

# 3. Show Grafana dashboard
# Open: http://localhost:3000
# Open imported dashboard → show CPU/memory graphs

# 4. Generate traffic to show live spikes
for ($i=0; $i -lt 20; $i++) { Invoke-WebRequest -Uri "http://localhost:9082" | Out-Null }
# Refresh Grafana → graphs spike
```