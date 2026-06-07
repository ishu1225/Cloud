# DOCKER + MINIKUBE + KUBERNETES — Viva + Exam Notes

---

## DOCKER — CORE CONCEPTS

|Term|Meaning|
|---|---|
|Image|Read-only template to create containers|
|Container|Running instance of an image|
|Dockerfile|Instructions to build an image|
|Docker Hub|Public registry to store/share images|
|Volume|Persistent storage for containers|
|Network|Communication channel between containers|
|Port Mapping|Expose container port to host (-p host:container)|

---

## DOCKERFILE — WRITE FROM SCRATCH IN EXAM

```dockerfile
FROM python:3.11-slim          # base image
WORKDIR /app                   # set working directory
RUN pip install flask           # run command during build
COPY app.py .                  # copy file into image
EXPOSE 5000                    # document port (not publish)
CMD ["python", "app.py"]       # command to run container
```

Minimal Flask app (`app.py`):

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "DevOps Lab - zen1tsu", 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

## SCENARIO 1 — Build image, run container, verify

```powershell
# Build image from Dockerfile in current folder
docker build -t zen1tsu/devops-app:v1 .

# Run container with port mapping
docker run -d --name my-app -p 8080:5000 zen1tsu/devops-app:v1

# Verify app is running
curl http://localhost:8080

# Check container status
docker ps

# Check logs
docker logs my-app
```

---

## SCENARIO 2 — Pull image from Hub and run

```powershell
# Pull specific image
docker pull zen1tsu/devops-app:v1

# Run it
docker run -d --name exam-app -p 8080:5000 zen1tsu/devops-app:v1

# Verify
curl http://localhost:8080
```

---

## SCENARIO 3 — Container lifecycle

```powershell
docker stop my-app          # stop running container
docker start my-app         # start stopped container
docker restart my-app       # restart container
docker rm my-app            # remove stopped container
docker rm -f my-app         # force remove running container
docker ps                   # list running containers
docker ps -a                # list all containers including stopped
```

---

## SCENARIO 4 — Push image to Docker Hub

```powershell
# Login
docker login

# Tag image with your Hub username
docker tag devops-app:v1 zen1tsu/devops-app:v1

# Push
docker push zen1tsu/devops-app:v1

# Also push latest tag
docker tag devops-app:v1 zen1tsu/devops-app:latest
docker push zen1tsu/devops-app:latest
```

---

## SCENARIO 5 — Nginx container (quick demo)

```powershell
docker run -d --name nginx-demo -p 9081:80 nginx:alpine
curl http://localhost:9081
docker logs nginx-demo
docker rm -f nginx-demo
```

---

## ALL IMPORTANT DOCKER COMMANDS

```powershell
docker build -t <name>:<tag> .          # build image
docker images                           # list images
docker rmi <image>                      # remove image
docker pull <image>                     # pull from Hub
docker push <image>                     # push to Hub
docker run -d -p <h>:<c> --name <n> <img>  # run container
docker ps                               # running containers
docker ps -a                            # all containers
docker stop <name>                      # stop
docker start <name>                     # start
docker restart <name>                   # restart
docker rm -f <name>                     # force remove
docker logs <name>                      # view logs
docker exec -it <name> bash             # enter container
docker inspect <name>                   # detailed info
docker network create <name>            # create network
docker network ls                       # list networks
docker volume create <name>             # create volume
docker volume ls                        # list volumes
docker stats                            # live resource usage
```

---

## KUBERNETES — CORE CONCEPTS

|Term|Meaning|
|---|---|
|Pod|Smallest unit — runs one or more containers|
|Deployment|Manages Pods, handles rollouts and scaling|
|Service|Exposes Pods to network (NodePort = external access)|
|ReplicaSet|Ensures desired number of Pod copies running|
|ConfigMap|Store non-secret config data|
|Namespace|Logical isolation within cluster|
|kubectl|CLI tool to interact with Kubernetes|
|Minikube|Single-node local Kubernetes cluster|

---

## K8S FILES — WRITE FROM SCRATCH IN EXAM

### deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-deploy
  labels:
    app: devops-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-app
  template:
    metadata:
      labels:
        app: devops-app
    spec:
      containers:
        - name: devops-app
          image: zen1tsu/devops-app:v1
          ports:
            - containerPort: 5000
```

### service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devops-service
spec:
  type: NodePort
  selector:
    app: devops-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30082
```

### configmap.yml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: devops-config
data:
  APP_ENV: "production"
  APP_NAME: "devops-lab"
```

---

## SCENARIO 6 — Full Minikube deployment from scratch

```powershell
# Start Minikube
minikube start --driver=docker

# Verify cluster is running
kubectl get nodes
# NAME       STATUS   ROLES           AGE
# minikube   Ready    control-plane   Xm

# Apply deployment and service
kubectl apply -f k8s\deployment.yml
kubectl apply -f k8s\service.yml

# Watch pods come up
kubectl get pods -w
# NAME                             READY   STATUS    
# devops-deploy-xxx                1/1     Running
# devops-deploy-xxx                1/1     Running

# Get service URL and test
minikube service devops-service --url
curl http://127.0.0.1:<PORT>
```

---

## SCENARIO 7 — Pull image already on Hub, run as Pod

```powershell
# Direct pod run (no deployment file needed)
kubectl run devops-pod --image=zen1tsu/devops-app:v1 --port=5000

# Verify
kubectl get pods
kubectl describe pod devops-pod
kubectl logs devops-pod

# Delete pod
kubectl delete pod devops-pod
```

---

## SCENARIO 8 — Scale replicas

```powershell
kubectl scale deployment devops-deploy --replicas=3
kubectl get pods        # shows 3 pods

kubectl scale deployment devops-deploy --replicas=1
kubectl get pods        # back to 1
```

---

## SCENARIO 9 — Update image version (rolling update)

```powershell
# Update to new version
kubectl set image deployment/devops-deploy devops-app=zen1tsu/devops-app:v2

# Watch rollout — old pods terminate, new pods start
kubectl rollout status deployment/devops-deploy

# See rollout history
kubectl rollout history deployment/devops-deploy

# Rollback if something breaks
kubectl rollout undo deployment/devops-deploy
```

---

## SCENARIO 10 — Remove old pods and redeploy

```powershell
# Delete deployment (removes all pods)
kubectl delete deployment devops-deploy

# Delete service
kubectl delete service devops-service

# Redeploy fresh
kubectl apply -f k8s\deployment.yml
kubectl apply -f k8s\service.yml
kubectl get pods -w
```

---

## ALL IMPORTANT KUBECTL COMMANDS

```powershell
minikube start --driver=docker          # start cluster
minikube stop                           # stop cluster
minikube status                         # check status
minikube service <svc> --url            # get service URL
minikube ssh                            # SSH into minikube node

kubectl get nodes                       # cluster nodes
kubectl get pods                        # list pods
kubectl get pods -w                     # watch pods live
kubectl get deployments                 # list deployments
kubectl get svc                         # list services
kubectl get all                         # show everything

kubectl apply -f <file.yml>             # create/update resource
kubectl delete -f <file.yml>            # delete resource
kubectl delete pod <name>               # delete specific pod
kubectl delete deployment <name>        # delete deployment

kubectl describe pod <name>             # detailed pod info
kubectl logs <pod>                      # pod logs
kubectl exec -it <pod> -- bash          # enter pod shell

kubectl scale deployment <name> --replicas=3   # scale
kubectl set image deployment/<name> <container>=<image>:<tag>
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout history deployment/<name>
```

---

## COMMON PROBLEMS AND FIXES

|Problem|Cause|Fix|
|---|---|---|
|ImagePullBackOff|Can't pull image from Hub|Check image name/tag, ensure pushed to Hub|
|CrashLoopBackOff|Container crashes on start|Check `kubectl logs <pod>`|
|Pending pods|Not enough resources|`kubectl describe pod` to see reason|
|Connection refused|Minikube stopped|`minikube start --driver=docker`|
|kubectl can't connect|Kubeconfig outdated|Copy fresh `.kube\config`|

---

## VIVA QUESTIONS

**Q: Difference between Pod and Deployment?** A: Pod is a single running instance. Deployment manages multiple Pods, handles rolling updates, scaling and self-healing — if a Pod dies, Deployment restarts it.

**Q: What is NodePort service?** A: Exposes the service on a static port on every node. Range is 30000-32767. Allows external access to Pods.

**Q: What happens when you delete a Pod managed by a Deployment?** A: Deployment automatically creates a new Pod to maintain desired replica count.

**Q: Difference between docker run and kubectl apply?** A: `docker run` starts a single container on local Docker. `kubectl apply` creates Kubernetes resources (Pods, Deployments, Services) inside the cluster which manages them automatically.

**Q: What is a ReplicaSet?** A: It ensures a specified number of Pod replicas are running at all times. Deployment creates and manages ReplicaSet automatically.

**Q: What is the use of labels and selectors in K8s?** A: Labels are key-value pairs on Pods. Selectors in Service/Deployment use labels to find which Pods to target. This is how Service knows which Pods to route traffic to.

**Q: How does Minikube differ from a real K8s cluster?** A: Minikube is a single-node cluster for local development. Real clusters have multiple master and worker nodes with high availability.

---

## EXAM TIP — If examiner says "deploy an app to Minikube"

```powershell
# 1. Start minikube
minikube start --driver=docker

# 2. Write deployment.yml and service.yml (show above templates)

# 3. Apply
kubectl apply -f deployment.yml
kubectl apply -f service.yml

# 4. Verify
kubectl get pods
kubectl get svc
minikube service devops-service --url

# 5. Show scaling
kubectl scale deployment devops-deploy --replicas=3
kubectl get pods
```