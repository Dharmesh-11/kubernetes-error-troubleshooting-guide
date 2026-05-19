# Kubernetes Errors & Troubleshooting Guide

## Introduction
This README explains common Kubernetes errors, causes, troubleshooting, and fixes.

---

# Important Commands

| Command | Purpose |
|---|---|
| kubectl get pods | Check pod status |
| kubectl describe pod <pod-name> | Detailed pod info |
| kubectl logs <pod-name> | View logs |
| kubectl get events | Cluster events |
| kubectl get nodes | Node status |

---

# Common Kubernetes Errors

# 1. CrashLoopBackOff Error in Kubernetes

## What is CrashLoopBackOff?

`CrashLoopBackOff` means:

- Container starts
- Then crashes immediately
- Kubernetes restarts it again and again
- After multiple failures, Kubernetes waits before restarting

So the pod enters a **restart loop**.

Example:

```bash
STATUS: CrashLoopBackOff
```

---

# Common Causes and Fixes

## 1. Application Crash

### Cause

Application code has errors.

Examples:
- Syntax error
- Missing package
- Wrong configuration

### Fix

Check logs:

```bash
kubectl logs <pod-name>
```

Fix the application error and rebuild the Docker image.

---

## 2. Wrong Command or Entrypoint

### Cause

Container starts with wrong command.

Examples:
- Wrong file name
- Wrong startup command

### Fix

Check Deployment YAML:

```yaml
command: ["python", "app.py"]
```

Make sure:
- File exists
- Command is correct

---

## 3. Missing Environment Variables

### Cause

Required environment variables are missing.

Examples:
- Database host missing
- API key missing

### Fix

Add correct environment variables:

```yaml
env:
  - name: DB_HOST
    value: mysql-service
```

---

## 4. Database or Service Connection Failure

### Cause

Application cannot connect to:
- MySQL
- Redis
- Backend API

### Fix

Check:
- Service name
- Port
- Network connectivity

Verify the service is running properly.

---

## 5. Liveness Probe Failure

### Cause

Health check fails repeatedly.

Kubernetes kills the container because it thinks the app is unhealthy.

### Fix

Correct probe configuration:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

Increase startup delay if the app starts slowly:

```yaml
initialDelaySeconds: 30
```

---

## 6. Out Of Memory (OOMKilled)

### Cause

Container uses more memory than the configured limit.

Kubernetes kills the container.

### Fix

Increase memory limits:

```yaml
resources:
  limits:
    memory: "512Mi"
```

---

## 7. Wrong or Corrupted Docker Image

### Cause

Docker image is built incorrectly.

### Fix

Rebuild and push the image again:

```bash
docker build -t myapp:v1 .
docker push myapp:v1
```

Update the deployment after pushing the image.

---

# Main Troubleshooting Command

```bash
kubectl logs <pod-name>
```

This command usually shows the real reason for the `CrashLoopBackOff` error.

---

---

# 2. ImagePullBackOff

# ImagePullBackOff Error in Kubernetes

## What is ImagePullBackOff?

`ImagePullBackOff` means Kubernetes is unable to download the container image from the container registry.

Kubernetes tries to pull the image multiple times, but after repeated failures, it stops temporarily and shows:

```bash
STATUS: ImagePullBackOff
```

---

# Common Causes and Fixes

## 1. Wrong Image Name

### Cause

The image name is incorrect.

Example:

```yaml
image: nginxh:latest
```

`nginxh` is wrong.

### Fix

Use the correct image name:

```yaml
image: nginx:latest
```

---

## 2. Wrong Image Tag

### Cause

The specified image tag does not exist.

Example:

```yaml
image: myapp:v5
```

But only `v1` exists.

### Fix

Check available tags and use the correct one:

```yaml
image: myapp:v1
```

---

## 3. Image Not Pushed to Docker Hub

### Cause

Image exists locally but is not uploaded to Docker Hub or another registry.

### Fix

Push the image:

```bash
docker push username/myapp:v1
```

Then update deployment YAML:

```yaml
image: username/myapp:v1
```

---

## 4. Private Docker Repository

### Cause

Kubernetes cannot access a private image repository without authentication.

### Fix

Create Docker registry secret:

```bash
kubectl create secret docker-registry regcred \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

Add secret in deployment:

```yaml
imagePullSecrets:
  - name: regcred
```

---

## 5. No Internet Connection on Node

### Cause

Kubernetes node cannot access Docker Hub or registry.

### Fix

Check internet connectivity on worker node:

```bash
ping google.com
```

Verify firewall and network settings.

---

## 6. Docker Hub Rate Limit

### Cause

Too many image pull requests from Docker Hub.

### Fix

Login to Docker Hub:

```bash
docker login
```

Or use another registry like:
- GitHub Container Registry
- AWS ECR
- Google Artifact Registry

---

## 7. Typo in Repository Name

### Cause

Repository name is incorrect.

Example:

```yaml
image: usernme/myapp:v1
```

### Fix

Correct repository name:

```yaml
image: username/myapp:v1
```

---

# Main Troubleshooting Commands

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section for exact error.

Example:

```bash
Failed to pull image
```

OR

```bash
pull access denied
```

---

# Example Error

```bash
Failed to pull image "myapp:v1"
```

### Fix

Push image properly:

```bash
docker push username/myapp:v1
```

Update deployment:

```yaml
image: username/myapp:v1
```

---

# Important Tip

Most `ImagePullBackOff` errors happen because of:
- Wrong image name
- Wrong tag
- Image not pushed
- Private repository access issue

---

---

# 3. ErrImagePull

# ErrImagePull Error in Kubernetes

## What is ErrImagePull?

`ErrImagePull` means Kubernetes failed to pull the container image from the container registry.

It happens when Kubernetes tries to download the image for the first time and fails immediately.

Example:

```bash
STATUS: ErrImagePull
```

After multiple retries, it may change to:

```bash
STATUS: ImagePullBackOff
```

---

# Common Causes and Fixes

## 1. Wrong Image Name

### Cause

Image name is incorrect.

Example:

```yaml
image: nginxx:latest
```

### Fix

Use correct image name:

```yaml
image: nginx:latest
```

---

## 2. Wrong Image Tag

### Cause

Specified tag does not exist.

Example:

```yaml
image: myapp:v10
```

But only `v1` exists.

### Fix

Use correct tag:

```yaml
image: myapp:v1
```

---

## 3. Image Not Pushed to Registry

### Cause

Image exists only on local machine.

Kubernetes cannot find it in Docker Hub or registry.

### Fix

Push image:

```bash
docker push username/myapp:v1
```

Update deployment YAML:

```yaml
image: username/myapp:v1
```

---

## 4. Private Repository Access Denied

### Cause

Image is stored in a private Docker repository.

Kubernetes has no authentication.

### Fix

Create Docker registry secret:

```bash
kubectl create secret docker-registry regcred \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

Add secret in deployment:

```yaml
imagePullSecrets:
  - name: regcred
```

---

## 5. Internet or Network Issue

### Cause

Kubernetes node cannot connect to Docker Hub or registry.

### Fix

Check internet connection on node:

```bash
ping google.com
```

Verify firewall and DNS settings.

---

## 6. Typo in Repository Name

### Cause

Wrong repository name.

Example:

```yaml
image: usernme/myapp:v1
```

### Fix

Correct repository name:

```yaml
image: username/myapp:v1
```

---

# Main Troubleshooting Commands

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section.

Example errors:

```bash
Failed to pull image
```

OR

```bash
pull access denied
```

---

# Example

## Error

```bash
ErrImagePull
```

### Root Cause

Image not available in Docker Hub.

### Fix

Push image:

```bash
docker push username/myapp:v1
```

Update deployment with correct image name.

---

# Important Tip

`ErrImagePull` happens during the first image pull failure.

If Kubernetes retries multiple times and still fails, the status changes to:

```bash
ImagePullBackOff
```
---

---

#  4. Pending Pod

# Pending Pod Error in Kubernetes

## What is Pending Pod?

`Pending` means the Pod is created successfully, but Kubernetes cannot schedule or start it on any node.

The Pod stays in waiting state.

Example:

```bash
STATUS: Pending
```

---

# Common Causes and Fixes

## 1. Insufficient CPU or Memory

### Cause

Cluster nodes do not have enough:
- CPU
- Memory (RAM)

for the Pod requirements.

### Fix

Check node resources:

```bash
kubectl describe nodes
```

Reduce resource requests or add more nodes.

Example:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
```

---

## 2. No Available Worker Nodes

### Cause

No worker node is ready to run the Pod.

### Fix

Check node status:

```bash
kubectl get nodes
```

If node is `NotReady`, fix the node issue or add new worker nodes.

---

## 3. Persistent Volume Not Available

### Cause

Pod is waiting for storage.

PVC is not bound to a PV.

### Fix

Check PVC status:

```bash
kubectl get pvc
```

If status is `Pending`, create a matching Persistent Volume.

---

## 4. Node Selector or Affinity Issue

### Cause

Pod requests a specific node label, but no matching node exists.

Example:

```yaml
nodeSelector:
  env: production
```

But nodes do not have that label.

### Fix

Check node labels:

```bash
kubectl get nodes --show-labels
```

Add correct label or update nodeSelector.

---

## 5. Taints and Tolerations Problem

### Cause

Node has taints and Pod has no matching toleration.

### Fix

Check node taints:

```bash
kubectl describe node <node-name>
```

Add toleration in Pod YAML:

```yaml
tolerations:
  - key: "app"
    operator: "Equal"
    value: "test"
    effect: "NoSchedule"
```

---

## 6. Image Pull Waiting

### Cause

Kubernetes is waiting to pull container image.

Sometimes Pod may stay Pending temporarily.

### Fix

Check image name and registry access.

Verify image exists.

---

## 7. Scheduler Problem

### Cause

Kubernetes scheduler is not working properly.

### Fix

Check scheduler Pod:

```bash
kubectl get pods -n kube-system
```

Restart scheduler if needed.

---

# Main Troubleshooting Commands

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section for exact reason.

Example:

```bash
0/2 nodes available: insufficient memory
```

OR

```bash
persistentvolumeclaim is not bound
```

---

## Check Nodes

```bash
kubectl get nodes
```

---

# Example

## Error

```bash
STATUS: Pending
```

### Root Cause

Not enough memory in cluster nodes.

### Fix

Reduce resource requests or add more worker nodes.

---

# Important Tip

Most `Pending` Pod issues happen because of:
- Insufficient CPU or RAM
- PVC storage issue
- Node selector mismatch
- Taints and tolerations
- No available worker node

---

---

# 5. OOMKilled

# OOMKilled Error in Kubernetes

## What is OOMKilled?

`OOMKilled` means:

The container used more memory than the limit assigned to it, so Kubernetes automatically killed the container.

OOM = Out Of Memory

Example:

```bash
Reason: OOMKilled
```

---

# Common Causes and Fixes

## 1. Memory Limit Too Low

### Cause

Application needs more RAM than the configured memory limit.

Example:

```yaml
resources:
  limits:
    memory: "128Mi"
```

But application uses 300Mi.

### Fix

Increase memory limit:

```yaml
resources:
  limits:
    memory: "512Mi"
```

---

## 2. Memory Leak in Application

### Cause

Application continuously consumes memory and never releases it.

Common in:
- Java apps
- Node.js apps
- Python apps

### Fix

Fix application code and optimize memory usage.

Restarting only gives temporary relief.

---

## 3. Large File or Data Processing

### Cause

Application loads very large files or datasets into memory.

### Fix

Process data in smaller chunks instead of loading everything at once.

---

## 4. Too Many Processes Running

### Cause

Application creates too many threads or processes.

### Fix

Limit thread/process creation and optimize application performance.

---

## 5. Wrong Resource Requests and Limits

### Cause

Memory requests and limits are configured incorrectly.

### Fix

Set proper resource values:

```yaml
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
```

---

# How to Troubleshoot OOMKilled

## Check Pod Details

```bash
kubectl describe pod <pod-name>
```

Look for:

```bash
Reason: OOMKilled
```

---

## Check Pod Logs

```bash
kubectl logs <pod-name>
```

Check if application crashes before termination.

---

## Check Resource Usage

```bash
kubectl top pod
```

Check memory consumption.

---

# Example

## Error

```bash
OOMKilled
```

### Root Cause

Container memory limit too small.

### Fix

Increase memory:

```yaml
resources:
  limits:
    memory: "1Gi"
```

Redeploy application.

---

# Important Tip

`OOMKilled` is one of the most common Kubernetes errors.

Most cases are fixed by:
- Increasing memory limits
- Optimizing application memory usage
- Setting proper resource requests and limits

---

---

# 6. CreateContainerConfigError

# CreateContainerConfigError in Kubernetes

## What is CreateContainerConfigError?

`CreateContainerConfigError` means Kubernetes cannot create the container because of a configuration problem.

The Pod is created, but the container cannot start due to invalid or missing configuration.

Example:

```bash
STATUS: CreateContainerConfigError
```

---

# Common Causes and Fixes

## 1. Missing ConfigMap

### Cause

Pod is trying to use a ConfigMap that does not exist.

Example:

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

But `app-config` is missing.

### Fix

Check ConfigMaps:

```bash
kubectl get configmap
```

Create missing ConfigMap:

```bash
kubectl create configmap app-config --from-literal=ENV=prod
```

---

## 2. Missing Secret

### Cause

Pod references a Secret that does not exist.

Example:

```yaml
secretKeyRef:
  name: db-secret
```

### Fix

Check Secrets:

```bash
kubectl get secrets
```

Create missing Secret:

```bash
kubectl create secret generic db-secret \
  --from-literal=password=admin123
```

---

## 3. Wrong ConfigMap or Secret Key

### Cause

ConfigMap or Secret exists, but the key name is wrong.

Example:

```yaml
key: username
```

But actual key is:

```yaml
user
```

### Fix

Check keys:

```bash
kubectl describe configmap <configmap-name>
```

OR

```bash
kubectl describe secret <secret-name>
```

Use correct key names.

---

## 4. Invalid Environment Variable Configuration

### Cause

Environment variables are configured incorrectly.

### Fix

Check YAML syntax carefully.

Example:

```yaml
env:
  - name: DB_HOST
    value: mysql-service
```

---

## 5. Invalid Volume Mount Configuration

### Cause

Volume or mount path configuration is incorrect.

### Fix

Check:
- Volume names
- Mount paths
- PVC names

Example:

```yaml
volumeMounts:
  - name: app-volume
    mountPath: /data
```

---

## 6. Wrong YAML Configuration

### Cause

Deployment YAML contains invalid configuration.

### Fix

Validate YAML:

```bash
kubectl apply -f deployment.yaml --dry-run=client
```

Fix syntax and configuration errors.

---

# How to Troubleshoot

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section for exact error.

Example:

```bash
configmap "app-config" not found
```

OR

```bash
secret "db-secret" not found
```

---

# Example

## Error

```bash
CreateContainerConfigError
```

### Root Cause

Missing Secret.

### Fix

Create Secret:

```bash
kubectl create secret generic db-secret \
  --from-literal=password=admin123
```

Restart Pod.

---

# Important Tip

Most `CreateContainerConfigError` issues happen because of:
- Missing ConfigMap
- Missing Secret
- Wrong environment variable configuration
- Invalid volume configuration
- YAML mistakes

---

---

# 7. ContainerCreating

# ContainerCreating Status in Kubernetes

## What is ContainerCreating?

`ContainerCreating` means Kubernetes is preparing and creating the container.

The Pod is not fully started yet.

Kubernetes is performing tasks like:
- Pulling Docker image
- Creating network
- Attaching storage
- Mounting volumes
- Starting container runtime

Example:

```bash
STATUS: ContainerCreating
```

---

# Common Causes and Fixes

## 1. Large Docker Image

### Cause

Docker image size is very large, so downloading takes time.

### Fix

Wait for image pull to complete or use smaller optimized images.

Example:
- Use `alpine` images when possible

---

## 2. Slow Internet or Registry Access

### Cause

Node cannot download image quickly from Docker Hub or registry.

### Fix

Check internet connectivity on node:

```bash
ping google.com
```

Verify registry access.

---

## 3. Image Pull Problem

### Cause

Image cannot be downloaded properly.

Sometimes `ContainerCreating` later changes to:
- `ErrImagePull`
- `ImagePullBackOff`

### Fix

Check image name and tag carefully.

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

---

## 4. Persistent Volume Mount Delay

### Cause

Kubernetes is waiting for storage attachment.

PVC or PV may have issues.

### Fix

Check PVC status:

```bash
kubectl get pvc
```

If PVC is pending, fix storage configuration.

---

## 5. Node Resource Problem

### Cause

Node has insufficient:
- CPU
- Memory
- Disk space

### Fix

Check node resources:

```bash
kubectl describe node <node-name>
```

Free resources or add new nodes.

---

## 6. Container Runtime Problem

### Cause

Docker or containerd service is not working properly on node.

### Fix

Check container runtime status on node:

```bash
systemctl status docker
```

OR

```bash
systemctl status containerd
```

Restart service if needed.

---

## 7. Network Plugin Issue

### Cause

CNI network plugin is not working properly.

### Fix

Check kube-system Pods:

```bash
kubectl get pods -n kube-system
```

Ensure network plugin Pods are running.

---

# How to Troubleshoot

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section.

Example:

```bash
Pulling image
```

OR

```bash
Unable to attach volume
```

---

## Check Node Status

```bash
kubectl get nodes
```

---

# Example

## Status

```bash
ContainerCreating
```

### Root Cause

Large Docker image downloading slowly.

### Fix

Wait for image pull or optimize image size.

---

# Important Tip

`ContainerCreating` is not always an error.

Sometimes Kubernetes simply needs time to:
- Pull image
- Attach storage
- Configure networking
- Start container

But if it stays too long, check:
- Image issues
- Storage issues
- Node resources
- Container runtime

---

---

# 8. Node NotReady

# Node NotReady Error in Kubernetes

## What is Node NotReady?

`NotReady` means the Kubernetes node is not healthy and cannot run Pods properly.

The control plane cannot communicate correctly with the node.

Example:

```bash
kubectl get nodes
```

Output:

```bash
NAME         STATUS
worker-1     NotReady
```

---

# Common Causes and Fixes

## 1. Kubelet Service Stopped

### Cause

`kubelet` service is not running on the node.

### Fix

Check kubelet status:

```bash
systemctl status kubelet
```

Start kubelet:

```bash
sudo systemctl start kubelet
```

Enable kubelet:

```bash
sudo systemctl enable kubelet
```

---

## 2. Network Problem

### Cause

Node cannot communicate with Kubernetes control plane.

### Fix

Check network connectivity:

```bash
ping <master-node-ip>
```

Verify:
- Firewall rules
- Security groups
- Network configuration

---

## 3. High CPU or Memory Usage

### Cause

Node resources are exhausted.

Node becomes unstable.

### Fix

Check resource usage:

```bash
top
```

OR

```bash
free -m
```

Free resources or add more nodes.

---

## 4. Disk Space Full

### Cause

Node storage is full.

Kubernetes cannot function properly.

### Fix

Check disk usage:

```bash
df -h
```

Remove unused:
- Docker images
- Containers
- Logs

Example:

```bash
docker system prune -a
```

---

## 5. Container Runtime Failure

### Cause

Docker or containerd service is not working.

### Fix

Check runtime status:

```bash
systemctl status docker
```

OR

```bash
systemctl status containerd
```

Restart runtime:

```bash
sudo systemctl restart docker
```

---

## 6. CNI Network Plugin Failure

### Cause

Kubernetes networking plugin is not running correctly.

### Fix

Check kube-system Pods:

```bash
kubectl get pods -n kube-system
```

Verify network plugin Pods are running.

---

## 7. Node Lost Connection with Control Plane

### Cause

Worker node disconnected from master node.

### Fix

Restart kubelet and verify node connectivity.

Sometimes node reboot may help.

---

# How to Troubleshoot

## Check Node Status

```bash
kubectl get nodes
```

---

## Describe Node

```bash
kubectl describe node <node-name>
```

Check Conditions and Events.

---

## Check Kubelet Logs

```bash
journalctl -u kubelet -f
```

---

# Example

## Error

```bash
STATUS: NotReady
```

### Root Cause

Kubelet service stopped.

### Fix

Start kubelet:

```bash
sudo systemctl start kubelet
```

---

# Important Tip

Most `Node NotReady` issues happen because of:
- Kubelet stopped
- Network problems
- Disk full
- Memory or CPU exhaustion
- Container runtime failure
- CNI plugin issue

---

---

# 9. PVC Pending

# PVC Pending Error in Kubernetes

## What is PVC Pending?

`PVC Pending` means the Persistent Volume Claim (PVC) is waiting for storage and cannot find a matching Persistent Volume (PV).

The storage request is not fulfilled yet.

Example:

```bash
STATUS: Pending
```

---

# Common Causes and Fixes

## 1. No Persistent Volume Available

### Cause

No PV exists that matches the PVC request.

### Fix

Check available PVs:

```bash
kubectl get pv
```

Create a matching Persistent Volume.

Example:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```

---

## 2. Storage Class Not Found

### Cause

PVC requests a StorageClass that does not exist.

Example:

```yaml
storageClassName: fast-storage
```

But `fast-storage` is missing.

### Fix

Check StorageClasses:

```bash
kubectl get storageclass
```

Use correct StorageClass name.

---

## 3. Storage Size Mismatch

### Cause

PVC requests more storage than available in PV.

Example:

PVC requests:

```yaml
storage: 10Gi
```

But PV has only:

```yaml
storage: 5Gi
```

### Fix

Create larger PV or reduce PVC request size.

---

## 4. Access Mode Mismatch

### Cause

PVC access mode does not match PV access mode.

Example:

PVC:

```yaml
ReadWriteMany
```

PV:

```yaml
ReadWriteOnce
```

### Fix

Ensure both use compatible access modes.

---

## 5. Dynamic Provisioning Not Working

### Cause

Storage provisioner is not working properly.

### Fix

Check StorageClass and provisioner Pods.

Example:

```bash
kubectl get pods -n kube-system
```

Verify storage provisioner is running.

---

## 6. Wrong Storage Configuration

### Cause

Incorrect PVC YAML configuration.

### Fix

Validate YAML carefully.

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

# How to Troubleshoot

## Check PVC Status

```bash
kubectl get pvc
```

---

## Describe PVC

```bash
kubectl describe pvc <pvc-name>
```

Check Events section.

Example:

```bash
no persistent volumes available
```

OR

```bash
storageclass not found
```

---

## Check Persistent Volumes

```bash
kubectl get pv
```

---

## Check Storage Classes

```bash
kubectl get storageclass
```

---

# Example

## Error

```bash
PVC STATUS: Pending
```

### Root Cause

No matching Persistent Volume available.

### Fix

Create a PV with:
- Correct storage size
- Correct access mode
- Matching StorageClass

---

# Important Tip

Most `PVC Pending` issues happen because of:
- Missing PV
- Wrong StorageClass
- Storage size mismatch
- Access mode mismatch
- Storage provisioner issue

---

---

# 10. Readiness Probe Failed

# Readiness Probe Failed in Kubernetes

## What is Readiness Probe Failed?

`Readiness Probe Failed` means Kubernetes thinks the application is not ready to receive traffic.

The Pod is running, but Kubernetes removes it from the Service endpoints until it becomes healthy.

Example:

```bash
Readiness probe failed
```

---

# Common Causes and Fixes

## 1. Wrong Probe Path

### Cause

The probe path does not exist in the application.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
```

But `/health` endpoint is missing.

### Fix

Use the correct application endpoint.

Example:

```yaml
path: /ready
```

---

## 2. Wrong Port Number

### Cause

Probe checks the wrong port.

Example:

```yaml
port: 8080
```

But application runs on port `3000`.

### Fix

Use correct application port.

Example:

```yaml
port: 3000
```

---

## 3. Application Starting Slowly

### Cause

Application needs more startup time.

Probe runs too early and fails.

### Fix

Increase delay time:

```yaml
initialDelaySeconds: 30
```

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
```

---

## 4. Application Crash or Internal Error

### Cause

Application has internal issues and cannot respond properly.

### Fix

Check application logs:

```bash
kubectl logs <pod-name>
```

Fix application errors.

---

## 5. Database or Backend Dependency Failure

### Cause

Application depends on:
- Database
- API
- Redis
- Backend service

If dependency fails, readiness probe also fails.

### Fix

Verify dependent services are running and reachable.

---

## 6. Timeout Too Short

### Cause

Application responds slowly, but probe timeout is too small.

### Fix

Increase timeout:

```yaml
timeoutSeconds: 10
```

---

## 7. Incorrect Probe Type

### Cause

Wrong probe type used.

Examples:
- HTTP probe for TCP application
- TCP probe for HTTP app

### Fix

Use correct probe type:
- `httpGet`
- `tcpSocket`
- `exec`

---

# Example Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

---

# How to Troubleshoot

## Check Pod Details

```bash
kubectl describe pod <pod-name>
```

Check Events section.

Example:

```bash
Readiness probe failed: HTTP probe failed with statuscode: 500
```

---

## Check Application Logs

```bash
kubectl logs <pod-name>
```

---

## Test Endpoint Inside Pod

```bash
kubectl exec -it <pod-name> -- sh
```

Then test endpoint:

```bash
curl localhost:8080/health
```

---

# Example

## Error

```bash
Readiness probe failed
```

### Root Cause

Wrong health check endpoint.

### Fix

Update readiness probe path:

```yaml
path: /ready
```

---

# Important Tip

When readiness probe fails:
- Pod keeps running
- But traffic is NOT sent to the Pod

Most readiness probe issues happen because of:
- Wrong endpoint
- Wrong port
- Slow startup
- Dependency failure
- Application errors



---

---

# 11. Liveness Probe Failed

# Liveness Probe Failed in Kubernetes

## What is Liveness Probe Failed?

`Liveness Probe Failed` means Kubernetes thinks the application inside the container is dead or unhealthy.

Kubernetes automatically kills and restarts the container.

Example:

```bash
Liveness probe failed
```

---

# Common Causes and Fixes

## 1. Wrong Probe Path

### Cause

The health check endpoint does not exist.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
```

But `/health` endpoint is missing.

### Fix

Use the correct endpoint.

Example:

```yaml
path: /live
```

---

## 2. Wrong Port Number

### Cause

Probe checks the wrong application port.

Example:

```yaml
port: 8080
```

But application runs on `3000`.

### Fix

Use correct port:

```yaml
port: 3000
```

---

## 3. Application Starts Slowly

### Cause

Probe runs before application fully starts.

Kubernetes thinks app is unhealthy and restarts it.

### Fix

Increase startup delay:

```yaml
initialDelaySeconds: 30
```

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
```

---

## 4. Application Crash or Freeze

### Cause

Application hangs, crashes, or becomes unresponsive.

### Fix

Check logs:

```bash
kubectl logs <pod-name>
```

Fix application issues.

---

## 5. Database or Dependency Failure

### Cause

Application depends on:
- Database
- Redis
- Backend API

If dependency fails, app becomes unhealthy.

### Fix

Verify dependent services are working properly.

---

## 6. Timeout Too Short

### Cause

Application response is slow.

Probe timeout expires before response.

### Fix

Increase timeout:

```yaml
timeoutSeconds: 10
```

---

## 7. Incorrect Probe Type

### Cause

Wrong probe type is configured.

Examples:
- HTTP probe for TCP app
- TCP probe for HTTP app

### Fix

Use correct probe type:
- `httpGet`
- `tcpSocket`
- `exec`

---

# Example Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
```

---

# How to Troubleshoot

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section.

Example:

```bash
Liveness probe failed: HTTP probe failed with statuscode: 500
```

---

## Check Application Logs

```bash
kubectl logs <pod-name>
```

---

## Test Endpoint Inside Pod

```bash
kubectl exec -it <pod-name> -- sh
```

Then test endpoint:

```bash
curl localhost:8080/health
```

---

# Example

## Error

```bash
Liveness probe failed
```

### Root Cause

Application startup takes too long.

### Fix

Increase delay:

```yaml
initialDelaySeconds: 30
```

---

# Important Tip

When liveness probe fails:
- Kubernetes restarts the container automatically

Most liveness probe issues happen because of:
- Wrong endpoint
- Wrong port
- Slow startup
- Application crash
- Dependency failure
- Incorrect timeout



---

---

# 12. DNS Resolution Failure

# DNS Resolution Failure in Kubernetes

## What is DNS Resolution Failure?

`DNS Resolution Failure` means a Pod cannot resolve domain names or Kubernetes service names into IP addresses.

The application cannot communicate with other services using names.

Example:

```bash
nslookup mysql-service
```

Output:

```bash
server can't find mysql-service
```

---

# Common Causes and Fixes

## 1. CoreDNS Pod Not Running

### Cause

Kubernetes DNS service (`CoreDNS`) is stopped or failed.

### Fix

Check CoreDNS Pods:

```bash
kubectl get pods -n kube-system
```

Restart CoreDNS if needed:

```bash
kubectl rollout restart deployment coredns -n kube-system
```

---

## 2. Wrong Service Name

### Cause

Application is using an incorrect Kubernetes Service name.

Example:

```bash
mysql-servce
```

Correct name:

```bash
mysql-service
```

### Fix

Check Services:

```bash
kubectl get svc
```

Use correct service name.

---

## 3. Service Does Not Exist

### Cause

Requested Kubernetes Service is missing.

### Fix

Create the Service or use the correct service name.

Check services:

```bash
kubectl get svc
```

---

## 4. Network Plugin Issue

### Cause

CNI network plugin is not working properly.

Pods cannot communicate internally.

### Fix

Check kube-system Pods:

```bash
kubectl get pods -n kube-system
```

Verify network plugin Pods are running.

---

## 5. DNS Configuration Problem

### Cause

Incorrect DNS settings inside Pod.

### Fix

Check DNS configuration:

```bash
cat /etc/resolv.conf
```

Verify nameserver entries are correct.

---

## 6. Firewall or Network Blocking

### Cause

Firewall rules block DNS communication.

### Fix

Ensure port `53` is open for DNS traffic.

Check network policies and firewall rules.

---

## 7. Namespace Issue

### Cause

Service exists in another namespace.

Example:

```bash
mysql-service
```

But service exists in `dev` namespace.

### Fix

Use full DNS name:

```bash
mysql-service.dev.svc.cluster.local
```

---

# How to Troubleshoot

## Check CoreDNS Pods

```bash
kubectl get pods -n kube-system
```

---

## Check Services

```bash
kubectl get svc
```

---

## Test DNS Inside Pod

```bash
kubectl exec -it <pod-name> -- sh
```

Run:

```bash
nslookup kubernetes.default
```

OR

```bash
ping mysql-service
```

---

## Check DNS Logs

```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

# Example

## Error

```bash
Temporary failure in name resolution
```

### Root Cause

CoreDNS Pod not running.

### Fix

Restart CoreDNS:

```bash
kubectl rollout restart deployment coredns -n kube-system
```

---

# Important Tip

Most `DNS Resolution Failure` issues happen because of:
- CoreDNS failure
- Wrong Service name
- Missing Service
- Network plugin problem
- Namespace mismatch
- Firewall blocking DNS

---

---

# 13. Service Not Accessible

# Service Not Accessible in Kubernetes

## What is Service Not Accessible?

`Service Not Accessible` means the application or Kubernetes Service cannot be reached from:
- Browser
- Another Pod
- External users
- Internal cluster communication

The Service exists, but traffic is not reaching the application.

---

# Common Causes and Fixes

## 1. Wrong Service Type

### Cause

Incorrect Service type is configured.

Example:
- Using `ClusterIP` for external access

### Fix

Use correct Service type:

```yaml
type: NodePort
```

OR

```yaml
type: LoadBalancer
```

---

## 2. Selector Labels Mismatch

### Cause

Service selector does not match Pod labels.

Example:

Service selector:

```yaml
selector:
  app: nginx
```

Pod label:

```yaml
labels:
  app: web
```

### Fix

Ensure Service selector matches Pod labels exactly.

---

## 3. Pod Not Running

### Cause

Backend Pods are:
- Crashed
- Pending
- NotReady

### Fix

Check Pod status:

```bash
kubectl get pods
```

Fix Pod issues first.

---

## 4. Wrong Target Port

### Cause

Service forwards traffic to wrong container port.

Example:

```yaml
targetPort: 8080
```

But application runs on:

```yaml
3000
```

### Fix

Use correct targetPort.

Example:

```yaml
targetPort: 3000
```

---

## 5. Application Not Listening on Correct Port

### Cause

Application inside container is listening on different port.

### Fix

Verify application port configuration.

Check inside Pod:

```bash
kubectl exec -it <pod-name> -- sh
```

Run:

```bash
netstat -tulnp
```

---

## 6. Network Policy Blocking Traffic

### Cause

Kubernetes NetworkPolicy blocks communication.

### Fix

Check NetworkPolicies:

```bash
kubectl get networkpolicy
```

Allow required traffic.

---

## 7. NodePort or Firewall Blocked

### Cause

Firewall or cloud security group blocks Service port.

### Fix

Open required ports:
- NodePort range: `30000-32767`
- Application port

Verify firewall and security group rules.

---

## 8. DNS Resolution Problem

### Cause

Service name cannot resolve inside cluster.

### Fix

Check DNS:

```bash
nslookup <service-name>
```

Verify CoreDNS is running.

---

# How to Troubleshoot

## Check Services

```bash
kubectl get svc
```

---

## Describe Service

```bash
kubectl describe svc <service-name>
```

---

## Check Endpoints

```bash
kubectl get endpoints
```

If endpoints are empty, selector labels are wrong.

---

## Test Service Inside Pod

```bash
kubectl exec -it <pod-name> -- sh
```

Run:

```bash
curl <service-name>:<port>
```

---

# Example

## Problem

Service not opening in browser.

### Root Cause

Wrong selector labels.

### Fix

Update Service selector:

```yaml
selector:
  app: web
```

---

# Important Tip

Most `Service Not Accessible` issues happen because of:
- Wrong Service type
- Label mismatch
- Wrong targetPort
- Pod failure
- Firewall blocking
- NetworkPolicy issue
- DNS problem



---

---

# 14. FailedScheduling

# FailedScheduling Error in Kubernetes

## What is FailedScheduling?

`FailedScheduling` means Kubernetes scheduler cannot place the Pod on any worker node.

The Pod remains in `Pending` state because no suitable node is available.

Example:

```bash
Warning  FailedScheduling
```

---

# Common Causes and Fixes

## 1. Insufficient CPU Resources

### Cause

Cluster nodes do not have enough CPU for the Pod.

Example:

```bash
0/2 nodes available: insufficient cpu
```

### Fix

Reduce CPU request or add more nodes.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
```

---

## 2. Insufficient Memory Resources

### Cause

Nodes do not have enough RAM.

Example:

```bash
0/2 nodes available: insufficient memory
```

### Fix

Reduce memory requests or increase node memory.

Example:

```yaml
resources:
  requests:
    memory: "128Mi"
```

---

## 3. Node Selector Mismatch

### Cause

Pod requests a node label that does not exist.

Example:

```yaml
nodeSelector:
  env: production
```

But nodes do not have that label.

### Fix

Check node labels:

```bash
kubectl get nodes --show-labels
```

Add correct label or update nodeSelector.

---

## 4. Taints and Tolerations Issue

### Cause

Node has taints and Pod has no matching toleration.

### Fix

Check node taints:

```bash
kubectl describe node <node-name>
```

Add toleration:

```yaml
tolerations:
  - key: "app"
    operator: "Equal"
    value: "test"
    effect: "NoSchedule"
```

---

## 5. Persistent Volume Issue

### Cause

Required storage is not available.

PVC may still be pending.

### Fix

Check PVC:

```bash
kubectl get pvc
```

Fix storage issue before scheduling.

---

## 6. Node NotReady

### Cause

Worker nodes are unhealthy.

### Fix

Check node status:

```bash
kubectl get nodes
```

Fix nodes showing `NotReady`.

---

## 7. Pod Affinity or Anti-Affinity Rules

### Cause

Affinity rules prevent Pod scheduling.

### Fix

Check affinity configuration in Pod YAML.

Correct or remove invalid affinity rules.

---

# How to Troubleshoot

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section.

Example:

```bash
0/3 nodes available: insufficient memory
```

OR

```bash
node(s) didn't match node selector
```

---

## Check Node Resources

```bash
kubectl describe nodes
```

---

## Check Node Labels

```bash
kubectl get nodes --show-labels
```

---

# Example

## Error

```bash
FailedScheduling
```

### Root Cause

No node has enough memory.

### Fix

Reduce memory requests or add more worker nodes.

---

# Important Tip

Most `FailedScheduling` issues happen because of:
- Insufficient CPU
- Insufficient memory
- Node selector mismatch
- Taints and tolerations
- PVC issue
- Node NotReady
- Affinity rule problems


---

---

# 15. Evicted Pod
# Evicted Pod in Kubernetes

## What is Evicted Pod?

`Evicted` means Kubernetes removed the Pod from a node because the node was running out of resources.

Kubernetes automatically evicts Pods to protect node stability.

Example:

```bash
STATUS: Evicted
```

---

# Common Causes and Fixes

## 1. Low Memory on Node

### Cause

Node RAM usage becomes very high.

Kubernetes evicts Pods to free memory.

Example:

```bash
The node was low on resource: memory
```

### Fix

- Free memory on node
- Add more worker nodes
- Increase node RAM
- Optimize application memory usage

---

## 2. Disk Space Full

### Cause

Node storage becomes full.

Kubernetes evicts Pods to recover disk space.

### Fix

Check disk usage:

```bash
df -h
```

Remove unused:
- Docker images
- Containers
- Logs

Example:

```bash
docker system prune -a
```

---

## 3. High Ephemeral Storage Usage

### Cause

Pod uses too much temporary storage.

### Fix

Set ephemeral storage limits:

```yaml
resources:
  limits:
    ephemeral-storage: "1Gi"
```

Clean temporary files regularly.

---

## 4. Too Many Pods on Node

### Cause

Node capacity limit is reached.

### Fix

Add more worker nodes or distribute workloads properly.

---

## 5. Resource Requests Not Configured

### Cause

Pods run without proper CPU or memory requests.

Kubernetes cannot manage resources efficiently.

### Fix

Add resource requests and limits:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
```

---

# How to Troubleshoot

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section.

Example:

```bash
The node was low on resource: memory
```

OR

```bash
The node had condition: DiskPressure
```

---

## Check Node Resources

```bash
kubectl describe node <node-name>
```

---

## Check Disk Usage

```bash
df -h
```

---

# Example

## Error

```bash
STATUS: Evicted
```

### Root Cause

Node memory exhausted.

### Fix

Increase node memory or reduce application memory usage.

---

# Important Tip

Evicted Pods are usually caused by:
- Low memory
- Disk pressure
- High ephemeral storage usage
- Too many Pods on node
- Missing resource requests and limits

Evicted Pods do not restart automatically.
Usually you need to:
- Delete the evicted Pod
- Fix the resource issue
- Recreate the Pod

----

----

# 16. RunContainerError

# RunContainerError in Kubernetes

## What is RunContainerError?

`RunContainerError` means Kubernetes created the container, but failed to start or run it properly.

The container exists, but something prevents it from running successfully.

Example:

```bash
STATUS: RunContainerError
```

---

# Common Causes and Fixes

## 1. Wrong Startup Command

### Cause

Container command or entrypoint is incorrect.

Example:

```yaml
command: ["python", "app.py"]
```

But `app.py` does not exist.

### Fix

Verify:
- File exists
- Command is correct

Update Deployment YAML with correct command.

---

## 2. Missing Application File

### Cause

Required application files are missing inside container image.

### Fix

Check Docker image contents.

Rebuild image correctly:

```bash
docker build -t myapp:v1 .
```

Push updated image and redeploy.

---

## 3. Permission Denied Error

### Cause

Application file does not have execute permission.

Example:

```bash
permission denied
```

### Fix

Add execute permission:

```dockerfile
RUN chmod +x start.sh
```

---

## 4. Invalid Entrypoint Script

### Cause

Entrypoint script contains errors.

Examples:
- Wrong shell syntax
- Wrong file path

### Fix

Check startup script carefully.

Test script locally before building image.

---

## 5. Missing Dependency or Package

### Cause

Application requires packages that are not installed.

### Fix

Install dependencies properly.

Example:

```dockerfile
RUN npm install
```

OR

```dockerfile
RUN pip install -r requirements.txt
```

---

## 6. Volume Mount Problem

### Cause

Mounted volume path is incorrect or inaccessible.

### Fix

Check:
- Volume names
- Mount paths
- PVC configuration

---

## 7. Resource Limitation

### Cause

Container cannot start due to insufficient CPU or memory.

### Fix

Increase resource limits:

```yaml
resources:
  limits:
    memory: "512Mi"
```

---

# How to Troubleshoot

## Check Pod Status

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check Events section.

Example:

```bash
Error: failed to start container
```

OR

```bash
permission denied
```

---

## Check Container Logs

```bash
kubectl logs <pod-name>
```

---

# Example

## Error

```bash
RunContainerError
```

### Root Cause

Startup script missing execute permission.

### Fix

Add permission:

```dockerfile
RUN chmod +x start.sh
```

Rebuild and redeploy image.

---

# Important Tip

Most `RunContainerError` issues happen because of:
- Wrong startup command
- Missing files
- Permission issues
- Invalid entrypoint
- Missing dependencies
- Volume mount problems


---

---

# 17. DiskPressure/MemoryPressure

# DiskPressure and MemoryPressure in Kubernetes

# 1. DiskPressure

## What is DiskPressure?

`DiskPressure` means the Kubernetes node is running low on disk space.

The node does not have enough storage for:
- Containers
- Images
- Logs
- Temporary files

Example:

```bash
DiskPressure=True
```

---

# Common Causes and Fixes

## 1. Disk Space Full

### Cause

Node storage is almost full.

### Fix

Check disk usage:

```bash
df -h
```

Remove unused files and free space.

---

## 2. Too Many Docker Images

### Cause

Unused Docker images consume storage.

### Fix

Remove unused images:

```bash
docker system prune -a
```

---

## 3. Large Container Logs

### Cause

Container logs become very large.

### Fix

Clean logs:

```bash
truncate -s 0 /var/log/*.log
```

Configure log rotation.

---

## 4. High Ephemeral Storage Usage

### Cause

Pods use too much temporary storage.

### Fix

Set ephemeral storage limits:

```yaml
resources:
  limits:
    ephemeral-storage: "1Gi"
```

---

## 5. Too Many Containers or Pods

### Cause

Node stores too many containers and volumes.

### Fix

Delete unused Pods and containers.

Add more worker nodes if needed.

---

# How to Troubleshoot DiskPressure

## Check Node Status

```bash
kubectl describe node <node-name>
```

Look for:

```bash
DiskPressure=True
```

---

## Check Disk Usage

```bash
df -h
```

---

## Check Docker Usage

```bash
docker system df
```

---

# Example

## Error

```bash
Node has DiskPressure
```

### Root Cause

Disk storage full.

### Fix

Remove unused Docker images and logs.

---

---

# 2. MemoryPressure

## What is MemoryPressure?

`MemoryPressure` means the Kubernetes node is running low on RAM memory.

The node does not have enough available memory for Pods.

Example:

```bash
MemoryPressure=True
```

---

# Common Causes and Fixes

## 1. High Memory Usage

### Cause

Applications consume too much RAM.

### Fix

Check memory usage:

```bash
free -m
```

Optimize applications or increase node memory.

---

## 2. Memory Leak

### Cause

Application continuously consumes memory without releasing it.

### Fix

Fix application memory leak.

Restarting Pods gives only temporary relief.

---

## 3. Too Many Pods Running

### Cause

Node runs too many Pods and memory becomes exhausted.

### Fix

Distribute workloads across multiple nodes.

Add more worker nodes.

---

## 4. Missing Resource Limits

### Cause

Pods run without memory limits.

### Fix

Add memory requests and limits:

```yaml
resources:
  requests:
    memory: "128Mi"
  limits:
    memory: "512Mi"
```

---

## 5. Large Applications Running

### Cause

Heavy applications consume excessive memory.

### Fix

Increase node RAM or optimize application memory usage.

---

# How to Troubleshoot MemoryPressure

## Check Node Status

```bash
kubectl describe node <node-name>
```

Look for:

```bash
MemoryPressure=True
```

---

## Check Memory Usage

```bash
free -m
```

OR

```bash
top
```

---

## Check Pod Resource Usage

```bash
kubectl top pod
```

---

# Example

## Error

```bash
Node has MemoryPressure
```

### Root Cause

Node RAM exhausted.

### Fix

Add more memory or reduce application memory usage.

---

# Important Tip

## DiskPressure happens because of:
- Full disk
- Large logs
- Too many images
- High temporary storage usage

## MemoryPressure happens because of:
- Low RAM
- Memory leaks
- Too many Pods
- Missing memory limits

These conditions can cause:
- Pod eviction
- Failed scheduling
- Node instability

---


---

# 18. NetworkUnavailable

# NetworkUnavailable in Kubernetes

## What is NetworkUnavailable?

`NetworkUnavailable` means the Kubernetes node network is not working properly.

The node cannot communicate correctly with:
- Other nodes
- Pods
- Services
- Kubernetes cluster network

Example:

```bash
NetworkUnavailable=True
```

---

# Common Causes and Fixes

## 1. CNI Plugin Not Running

### Cause

Container Network Interface (CNI) plugin is failed or missing.

Examples:
- Calico
- Flannel
- Weave
- Cilium

### Fix

Check CNI Pods:

```bash
kubectl get pods -n kube-system
```

Restart failed network Pods.

---

## 2. Network Plugin Installation Problem

### Cause

CNI plugin was not installed correctly.

### Fix

Reinstall the network plugin.

Example for Flannel:

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

---

## 3. Node Cannot Reach Cluster Network

### Cause

Worker node has network connectivity problems.

### Fix

Check node connectivity:

```bash
ping <master-node-ip>
```

Verify:
- Routing
- Firewall
- Network interfaces

---

## 4. Firewall Blocking Traffic

### Cause

Firewall blocks Kubernetes networking ports.

### Fix

Open required ports:
- 6443
- 10250
- CNI plugin ports

Check firewall rules.

---

## 5. Incorrect Pod CIDR Configuration

### Cause

Pod network CIDR configuration mismatch.

### Fix

Verify Pod CIDR settings in:
- kubeadm config
- CNI plugin config

Ensure both match correctly.

---

## 6. Container Runtime Issue

### Cause

Docker or containerd networking is not functioning properly.

### Fix

Restart container runtime:

```bash
sudo systemctl restart docker
```

OR

```bash
sudo systemctl restart containerd
```

---

## 7. Node Network Interface Problem

### Cause

Physical or virtual network interface is down.

### Fix

Check interfaces:

```bash
ip addr
```

Restart network service if needed.

---

# How to Troubleshoot

## Check Node Status

```bash
kubectl describe node <node-name>
```

Look for:

```bash
NetworkUnavailable=True
```

---

## Check CNI Pods

```bash
kubectl get pods -n kube-system
```

---

## Check Node Connectivity

```bash
ping <master-node-ip>
```

---

## Check Network Interfaces

```bash
ip addr
```

---

## Check CNI Logs

```bash
kubectl logs -n kube-system <cni-pod-name>
```

---

# Example

## Error

```bash
NetworkUnavailable=True
```

### Root Cause

Flannel CNI Pod crashed.

### Fix

Restart or reinstall Flannel network plugin.

---

# Important Tip

Most `NetworkUnavailable` issues happen because of:
- CNI plugin failure
- Incorrect network configuration
- Firewall blocking
- Pod CIDR mismatch
- Container runtime networking issue
- Node connectivity problems

This issue can cause:
- Pods unable to communicate
- Services not reachable
- DNS failures
- Cluster networking problems

---

---


# Golden Troubleshooting Flow

```text
kubectl get pods
↓
kubectl describe pod
↓
kubectl logs
↓
Check events/resources/network
↓
Fix issue
```

---

# Best Practices

- Use probes
- Define resource limits
- Monitor logs
- Validate YAML
- Use ConfigMaps and Secrets properly

# Conclusion

Kubernetes errors are common during container deployment and cluster management. Understanding these errors helps in quickly identifying problems and maintaining application availability.

This guide covered major Kubernetes issues such as:
- CrashLoopBackOff
- ImagePullBackOff
- ErrImagePull
- Pending Pods
- OOMKilled
- CreateContainerConfigError
- ContainerCreating
- Node NotReady
- PVC Pending
- Readiness Probe Failed
- Liveness Probe Failed
- DNS Resolution Failure
- Service Not Accessible
- FailedScheduling
- Evicted Pods
- RunContainerError
- DiskPressure
- MemoryPressure
- NetworkUnavailable

For every error, understanding:
- the root cause,
- troubleshooting steps,
- and proper fixes

is essential for effective Kubernetes administration and troubleshooting.

Regular monitoring, proper resource management, correct configuration, and strong networking setup can prevent most Kubernetes issues and improve cluster stability and performance.

---

# Author

**Dharmesh Panpatil**
