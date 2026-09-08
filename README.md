# Kubernetes & Minikube Hands-On Lab

A hands-on Kubernetes laboratory for **Ubuntu 24.04** and **Linux Mint**.

The purpose of this project is to install a local Kubernetes cluster using **Minikube** and manually create, inspect, and manage the following Kubernetes resources:

```text
Namespace
   ↓
Secret
   ↓
ConfigMap
   ↓
StorageClass
   ↓
PersistentVolume
   ↓
PersistentVolumeClaim
   ↓
Pod
   ↓
Service
   ↓
Deployment
   ↓
Ingress
   ↓
StatefulSet
   ↓
DaemonSet
```

The laboratory also includes a multi-node Minikube cluster and the **Headlamp** Kubernetes dashboard.

---

# 1. Prerequisites

The laboratory was designed for:

* Ubuntu 24.04
* Linux Mint
* Docker
* kubectl
* Minikube

Make sure virtualization is enabled on your system and that you have enough CPU, RAM, and disk space to run several Kubernetes workloads.

---

# 2. Install Minikube and kubectl

## 2.1 Install Required Packages

Update the package index:

```bash
sudo apt update
```

Install the required packages:

```bash
sudo apt install -y curl wget apt-transport-https ca-certificates
```

---

## 2.2 Install kubectl

Download the latest stable version:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install it:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

Optional detailed output:

```bash
kubectl version --client -o yaml
```

Check the installation path:

```bash
which kubectl
```

---

## 2.3 Install Minikube

Download the latest Minikube binary:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Install it:

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify:

```bash
minikube version
```

Check the installation path:

```bash
which minikube
```

---

# 3. Install Docker

The laboratory uses Docker as the Minikube driver.

Install Docker:

```bash
sudo apt update
sudo apt install -y docker.io
```

Enable and start Docker:

```bash
sudo systemctl enable --now docker
```

Check the service:

```bash
sudo systemctl status docker
```

Test Docker:

```bash
sudo docker run hello-world
```

---

## 3.1 Run Docker Without sudo

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Log out and log in again.

Alternatively, refresh the current shell:

```bash
newgrp docker
```

Verify:

```bash
docker ps
```

The command should work without `sudo`.

---

# 4. Create the Minikube Cluster

Start Minikube using Docker:

```bash
minikube start --driver=docker
```

Check the cluster status:

```bash
minikube status
```

Expected result:

```text
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Check the Kubernetes nodes:

```bash
kubectl get nodes
```

Expected result:

```text
NAME       STATUS   ROLES           ...
minikube   Ready    control-plane   ...
```

Check all system Pods:

```bash
kubectl get pods -A
```

Check cluster information:

```bash
kubectl cluster-info
```

Check the Minikube profile:

```bash
minikube profile list
```

Get the Minikube IP:

```bash
minikube ip
```

---

# 5. Cluster Information

The following commands answer the initial laboratory questions.

## Where is kubectl installed?

```bash
which kubectl
```

Example:

```text
/usr/local/bin/kubectl
```

The exact path may be different depending on the installation method.

---

## Where is Minikube installed?

```bash
which minikube
```

Example:

```text
/usr/local/bin/minikube
```

---

## Which Kubernetes version is running?

```bash
kubectl version
```

The output contains both the client and server versions.

The **Server Version** is the Kubernetes version running inside Minikube.

A more focused command is:

```bash
kubectl get nodes
```

or:

```bash
kubectl version --output=yaml
```

---

## Which container runtime is used?

Run:

```bash
kubectl get nodes -o wide
```

The output shows information about the node.

For detailed runtime information:

```bash
kubectl describe node minikube
```

Look for:

```text
Container Runtime Version:
```

When Minikube is running with the Docker driver, the node normally uses a Docker-based container runtime.

---

## What is the Minikube IP?

Run:

```bash
minikube ip
```

Example:

```text
192.168.49.2
```

The exact IP depends on the local Minikube environment.

---

# 6. Inspect the Initial Cluster

List all namespaces:

```bash
kubectl get namespaces
```

List all Pods in all namespaces:

```bash
kubectl get pods -A
```

List nodes:

```bash
kubectl get nodes -o wide
```

Inspect cluster information:

```bash
kubectl cluster-info
```

Inspect Minikube nodes:

```bash
minikube node list
```

Typical system namespaces include:

```text
default
kube-node-lease
kube-public
kube-system
```

Additional namespaces may be created when addons such as Ingress or Headlamp are enabled.

---

# 7. Add Worker Nodes

The initial Minikube cluster normally contains one control-plane node.

Add the first worker:

```bash
minikube node add --worker -p minikube --name worker-01
```

Add a second worker:

```bash
minikube node add --worker -p minikube --name worker-02
```

Check Minikube nodes:

```bash
minikube node list
```

Check Kubernetes nodes:

```bash
kubectl get nodes -o wide
```

Expected topology:

```text
NAME          STATUS   ROLES
minikube      Ready    control-plane
worker-01     Ready    <none>
worker-02     Ready    <none>
```

`<none>` means that the node is a regular worker node without a control-plane role.

---

# 8. Apply the Kubernetes Manifests

All manifests should be stored in the project directory.

Example project structure:

```text
.
├── README.md
├── namespaces.yaml
├── configmap.yaml
├── configmap-pod.yaml
├── configmap-volume-pod.yaml
├── secret.yaml
├── nginx-pod.yaml
├── nginx-service.yaml
├── web-deployment.yaml
├── web-service.yaml
├── web-ingress.yaml
├── storageclass.yaml
├── pv.yaml
├── pvc.yaml
├── storage-pod.yaml
├── redis-service.yaml
├── redis-statefulset.yaml
└── daemonset.yaml
```

Apply the manifests in the following order.

---

## 8.1 Namespace

```bash
kubectl apply -f namespaces.yaml
```

Verify:

```bash
kubectl get namespaces
```

Set `dev` as the default namespace:

```bash
kubectl config set-context --current --namespace=dev
```

Verify:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
```

---

## 8.2 ConfigMap

```bash
kubectl apply -f configmap.yaml
```

Verify:

```bash
kubectl get configmap -n dev
```

Detailed information:

```bash
kubectl describe configmap app-config -n dev
```

---

## 8.3 ConfigMap Test Pod

```bash
kubectl apply -f configmap-pod.yaml
```

Verify:

```bash
kubectl get pod configmap-test -n dev
```

Check environment variables:

```bash
kubectl exec -n dev configmap-test -- env
```

---

## 8.4 ConfigMap Volume Pod

```bash
kubectl apply -f configmap-volume-pod.yaml
```

Verify:

```bash
kubectl get pod configmap-volume-test -n dev
```

Check mounted files:

```bash
kubectl exec -n dev configmap-volume-test -- ls -la /etc/config/
```

Read the configuration:

```bash
kubectl exec -n dev configmap-volume-test -- cat /etc/config/APP_MESSAGE
```

---

# 9. Secret

Apply:

```bash
kubectl apply -f secret.yaml
```

Verify:

```bash
kubectl get secret -n dev
```

Detailed information:

```bash
kubectl describe secret app-secret -n dev
```

Inspect the YAML representation:

```bash
kubectl get secret app-secret -n dev -o yaml
```

Demonstrate Base64 encoding:

```bash
echo -n 'supersecret' | base64
```

Decode it:

```bash
echo -n 'c3VwZXJzZWNyZXQ=' | base64 -d
```

> **Security:** Kubernetes Secret data is Base64-encoded by default. Base64 is not encryption. Never commit real passwords, API keys, or tokens to Git.

---

# 10. Pod

Apply the NGINX Pod:

```bash
kubectl apply -f nginx-pod.yaml
```

Verify:

```bash
kubectl get pod nginx-pod -n dev
```

Detailed information:

```bash
kubectl describe pod nginx-pod -n dev
```

View logs:

```bash
kubectl logs nginx-pod -n dev
```

Inspect environment variables:

```bash
kubectl exec -n dev nginx-pod -- env
```

Inspect the ConfigMap volume:

```bash
kubectl exec -n dev nginx-pod -- ls -la /etc/config
```

Inspect the Secret volume:

```bash
kubectl exec -n dev nginx-pod -- ls -la /etc/secrets
```

---

# 11. Service

Apply the Service:

```bash
kubectl apply -f nginx-service.yaml
```

Verify:

```bash
kubectl get svc -n dev
```

Check endpoints:

```bash
kubectl get endpoints -n dev
```

Check EndpointSlices:

```bash
kubectl get endpointslices -n dev
```

Start a temporary curl Pod:

```bash
kubectl run curl \
  -n dev \
  --image=curlimages/curl \
  -it \
  --rm \
  -- sh
```

Inside the container:

```bash
curl http://nginx-service
```

This verifies that Kubernetes Service discovery works.

---

# 12. Deployment

Apply:

```bash
kubectl apply -f web-deployment.yaml
```

Verify the Deployment:

```bash
kubectl get deployment -n dev
```

Check ReplicaSets:

```bash
kubectl get replicasets -n dev
```

Check Pods:

```bash
kubectl get pods -n dev -o wide
```

Or check everything together:

```bash
kubectl get deployment,replicaset,pods -n dev
```

The Deployment should create three replicas:

```text
web-xxxxxxxxxx-xxxxx
web-xxxxxxxxxx-xxxxx
web-xxxxxxxxxx-xxxxx
```

---

# 13. Deployment Rolling Update

The initial Deployment uses:

```yaml
image: nginx:1.27
```

Change it to:

```yaml
image: nginx:1.28
```

Apply the updated manifest:

```bash
kubectl apply -f web-deployment.yaml
```

Monitor the rollout:

```bash
kubectl rollout status deployment/web -n dev
```

View rollout history:

```bash
kubectl rollout history deployment/web -n dev
```

Rollback:

```bash
kubectl rollout undo deployment/web -n dev
```

Verify:

```bash
kubectl rollout status deployment/web -n dev
```

---

# 14. Ingress

Enable the Minikube Ingress addon:

```bash
minikube addons enable ingress
```

Check the Ingress controller:

```bash
kubectl get pods -n ingress-nginx
```

Apply the application Service:

```bash
kubectl apply -f web-service.yaml
```

Verify:

```bash
kubectl get svc -n dev
```

Apply the Ingress:

```bash
kubectl apply -f web-ingress.yaml
```

Verify:

```bash
kubectl get ingress -n dev
```

Detailed information:

```bash
kubectl describe ingress web-ingress -n dev
```

Get the Minikube IP:

```bash
minikube ip
```

Add `demo.local` to `/etc/hosts`:

```bash
echo "$(minikube ip) demo.local" | sudo tee -a /etc/hosts
```

Test the application:

```bash
curl http://demo.local
```

Expected response is the default NGINX page.

---

# 15. StorageClass

Apply:

```bash
kubectl apply -f storageclass.yaml
```

Verify:

```bash
kubectl get storageclass
```

Detailed information:

```bash
kubectl describe storageclass demo-storage
```

The project demonstrates a custom StorageClass configuration.

The manually created PV/PVC example uses:

```text
storageClassName: manual
```

This intentionally separates the manual PV/PVC exercise from the custom `demo-storage` StorageClass.

---

# 16. PersistentVolume

Create the storage directory inside Minikube:

```bash
minikube ssh
```

Inside the Minikube node:

```bash
sudo mkdir -p /mnt/data/demo-pv
sudo chmod 777 /mnt/data/demo-pv
exit
```

Apply the PersistentVolume:

```bash
kubectl apply -f pv.yaml
```

Verify:

```bash
kubectl get pv
```

Detailed information:

```bash
kubectl describe pv demo-pv
```

The PV should have a capacity of:

```text
1Gi
```

and use:

```text
hostPath: /mnt/data/demo-pv
```

---

# 17. PersistentVolumeClaim

Apply:

```bash
kubectl apply -f pvc.yaml
```

Verify:

```bash
kubectl get pvc -n dev
```

Check the PV:

```bash
kubectl get pv
```

The PVC requests:

```text
500Mi
```

and should become:

```text
STATUS: Bound
```

The relationship is:

```text
PVC
 |
 v
PV
 |
 v
/mnt/data/demo-pv
```

---

# 18. Pod with Persistent Storage

Apply:

```bash
kubectl apply -f storage-pod.yaml
```

Verify:

```bash
kubectl get pod storage-test -n dev
```

Write data to the persistent volume:

```bash
kubectl exec -n dev storage-test -- \
  sh -c 'echo "Hello Kubernetes Storage" > /data/test.txt'
```

Read the file:

```bash
kubectl exec -n dev storage-test -- cat /data/test.txt
```

Expected:

```text
Hello Kubernetes Storage
```

Delete the Pod:

```bash
kubectl delete pod storage-test -n dev
```

Recreate it:

```bash
kubectl apply -f storage-pod.yaml
```

Read the file again:

```bash
kubectl exec -n dev storage-test -- cat /data/test.txt
```

The data should still exist because it is stored on the persistent volume rather than in the Pod's ephemeral filesystem.

---

# 19. StatefulSet

Apply the Redis headless Service:

```bash
kubectl apply -f redis-service.yaml
```

Verify:

```bash
kubectl get svc -n dev
```

Apply the StatefulSet:

```bash
kubectl apply -f redis-statefulset.yaml
```

Verify:

```bash
kubectl get statefulset -n dev
```

Check the Pods:

```bash
kubectl get pods -n dev
```

Expected Pods:

```text
redis-0
redis-1
```

Check the PVCs:

```bash
kubectl get pvc -n dev
```

The StatefulSet creates a separate PVC for each replica.

---

# 20. StatefulSet DNS

Start a temporary DNS test Pod:

```bash
kubectl run dns-test \
  -n dev \
  --image=busybox:1.36 \
  -it \
  --rm \
  -- sh
```

Inside the container:

```bash
nslookup redis
```

Test the first StatefulSet Pod:

```bash
nslookup redis-0.redis
```

Test the second Pod:

```bash
nslookup redis-1.redis
```

Test the full DNS name:

```bash
nslookup redis-0.redis.dev.svc.cluster.local
```

The StatefulSet provides stable Pod identities and DNS names.

---

# 21. DaemonSet

Apply:

```bash
kubectl apply -f daemonset.yaml
```

Verify:

```bash
kubectl get daemonset -n dev
```

Check the Pods:

```bash
kubectl get pods -n dev -o wide
```

A DaemonSet schedules one Pod on every eligible node.

Add another node if necessary:

```bash
minikube node add --worker
```

Check the nodes:

```bash
kubectl get nodes
```

Check DaemonSet Pods:

```bash
kubectl get pods -n dev -o wide
```

The new node should receive a new `node-agent` Pod automatically.

---

# 22. Headlamp

Enable Headlamp:

```bash
minikube addons enable headlamp
```

Check the Headlamp Pods:

```bash
kubectl get pods -n headlamp
```

Create a temporary authentication token:

```bash
kubectl create token headlamp --duration=24h -n headlamp
```

Save the token securely.

Start Headlamp:

```bash
minikube service headlamp -n headlamp
```

Minikube will display a local URL similar to:

```text
http://<minikube-ip>:<port>
```

Open the displayed URL in a browser and use the generated token to log in.

> **Important:** Never store the Headlamp token in `README.md`, Git, or any public repository.

---

# 23. Verify All Resources

At the end of the laboratory, run the following commands.

## Namespaces

```bash
kubectl get namespaces
```

## ConfigMaps

```bash
kubectl get configmaps -n dev
```

## Secrets

```bash
kubectl get secrets -n dev
```

## Pods

```bash
kubectl get pods -n dev -o wide
```

## Services

```bash
kubectl get services -n dev
```

## Deployments

```bash
kubectl get deployments -n dev
```

## ReplicaSets

```bash
kubectl get replicasets -n dev
```

## Ingress

```bash
kubectl get ingress -n dev
```

## StorageClasses

```bash
kubectl get storageclasses
```

## PersistentVolumes

```bash
kubectl get pv
```

## PersistentVolumeClaims

```bash
kubectl get pvc -n dev
```

## StatefulSets

```bash
kubectl get statefulsets -n dev
```

## DaemonSets

```bash
kubectl get daemonsets -n dev
```

## Nodes

```bash
kubectl get nodes -o wide
```

---

# 24. One-Command Overview

To get a quick overview of the application namespace:

```bash
kubectl get all -n dev
```

For storage:

```bash
kubectl get storageclass,pv,pvc
```

For workloads:

```bash
kubectl get deployment,statefulset,daemonset,pods -n dev
```

For networking:

```bash
kubectl get svc,ingress,endpoints,endpointslices -n dev
```

---

# 25. Final Architecture

After completing the laboratory, the environment should look approximately like this:

```text
                         Local Browser
                              |
                              v
                       demo.local
                              |
                              v
                         Ingress
                              |
                              v
                        web-service
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
              web Pod      web Pod      web Pod
                 \            |            /
                  +-----------+-----------+
                              |
                         Deployment


 ConfigMap ────────────────┐
                            │
 Secret ───────────────────┼──> Application Pods
                            │
                            │
 StorageClass               │
      │                     │
      ▼                     │
     PV ◄── PVC ◄───────────┘
      │
      ▼
 Minikube hostPath


                       StatefulSet
                            |
                  +---------+---------+
                  |                   |
                  v                   v
               redis-0             redis-1
                  |                   |
                PVC-0               PVC-1
                  \                   /
                   +-------+---------+
                           |
                    Headless Service
                           |
                    Kubernetes DNS


                       DaemonSet
                           |
                +----------+----------+
                |                     |
                v                     v
           node-agent            node-agent
           worker-01             worker-02
```

---

# 26. Laboratory Questions and Answers

## 1. Where is `kubectl` located?

Run:

```bash
which kubectl
```

With the installation used in this laboratory, it is normally:

```text
/usr/local/bin/kubectl
```

The exact location can vary depending on how `kubectl` was installed.

---

## 2. Where is Minikube located?

Run:

```bash
which minikube
```

With the installation used in this laboratory, it is normally:

```text
/usr/local/bin/minikube
```

---

## 3. Which Kubernetes version is running?

Run:

```bash
kubectl version
```

The important value is the **Server Version**, because it identifies the Kubernetes version running inside the Minikube cluster.

You can also use:

```bash
kubectl get nodes
```

The exact version depends on the Minikube release used to create the cluster.

---

## 4. Which container runtime is used by the node?

Run:

```bash
kubectl describe node minikube
```

Look for:

```text
Container Runtime Version:
```

Because the cluster was started with:

```bash
minikube start --driver=docker
```

the cluster uses a Docker-based container runtime environment.

The exact runtime/version should always be taken from the running node rather than assumed from the Minikube driver alone.

---

## 5. What is the Minikube IP?

Run:

```bash
minikube ip
```

The exact IP is assigned by the local Minikube environment and can differ between installations.

Example:

```text
192.168.49.2
```

Do not hard-code this value in manifests.

---

## 6. Which system Pods were created?

Run:

```bash
kubectl get pods -A
```

Minikube creates Kubernetes system components in namespaces such as:

```text
kube-system
```

Depending on the Minikube/Kubernetes version and enabled addons, you may see components for:

* CoreDNS;
* kube-proxy;
* API server;
* controller manager;
* scheduler;
* etcd;
* storage provisioners;
* networking components.

Ingress and Headlamp create additional workloads in their respective namespaces after their addons are enabled.

---

## 7. Which namespaces exist?

Run:

```bash
kubectl get namespaces
```

A fresh Kubernetes cluster normally contains namespaces such as:

```text
default
kube-node-lease
kube-public
kube-system
```

This laboratory additionally creates:

```text
dev
monitoring
```

After enabling addons, additional namespaces such as `ingress-nginx` and `headlamp` may also appear.

---

## 8. What is the difference between a control-plane node and a worker node?

The **control plane** runs the components responsible for managing the Kubernetes cluster, including the API server, scheduler, controller manager, and cluster state components.

A **worker node** runs application workloads.

In this laboratory:

```text
minikube  → control-plane
worker-01 → worker
worker-02 → worker
```

---

## 9. What happens when a new node is added with a DaemonSet?

A DaemonSet ensures that a Pod is scheduled on every eligible node.

Therefore, when a new node is added:

```text
New Node
   ↓
DaemonSet detects the node
   ↓
New DaemonSet Pod is scheduled
```

This is why the `node-agent` workload increases when additional nodes are added.

---

## 10. Why does the StatefulSet create `redis-0` and `redis-1`?

StatefulSets provide stable identities to their Pods.

With:

```yaml
replicas: 2
```

the Pods receive predictable names:

```text
redis-0
redis-1
```

These names remain stable even when Pods are recreated.

---

## 11. Why does each Redis Pod have its own PVC?

The StatefulSet uses `volumeClaimTemplates`.

Kubernetes creates a separate PVC for each StatefulSet replica:

```text
redis-0 → PVC
redis-1 → PVC
```

This allows each stateful replica to have its own persistent storage.

---

## 12. What is the purpose of a Headless Service?

A Headless Service uses:

```yaml
clusterIP: None
```

Instead of providing one virtual ClusterIP, it allows Kubernetes DNS to expose the individual Pods.

For the Redis StatefulSet, this enables names such as:

```text
redis-0.redis
redis-1.redis
```

This is useful for stateful applications that need stable network identities.

---

# 27. Cleanup

Delete the resources created during the laboratory.

```bash
kubectl delete -f daemonset.yaml
kubectl delete -f redis-statefulset.yaml
kubectl delete -f redis-service.yaml
kubectl delete -f storage-pod.yaml
kubectl delete -f pvc.yaml
kubectl delete -f pv.yaml
kubectl delete -f storageclass.yaml
kubectl delete -f web-ingress.yaml
kubectl delete -f web-service.yaml
kubectl delete -f web-deployment.yaml
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-pod.yaml
kubectl delete -f secret.yaml
kubectl delete -f configmap-volume-pod.yaml
kubectl delete -f configmap-pod.yaml
kubectl delete -f configmap.yaml
kubectl delete -f namespaces.yaml
```

If the `configmap-test` or `configmap-volume-test` Pods are still running, they can be removed with:

```bash
kubectl delete pod configmap-test configmap-volume-test -n dev
```

---

## Remove the Ingress Addon

```bash
minikube addons disable ingress
```

---

## Remove Headlamp

```bash
minikube addons disable headlamp
```

---

## Remove the Entire Minikube Cluster

To completely delete the local Kubernetes cluster:

```bash
minikube delete
```

Verify:

```bash
minikube status
```

---

# 28. Conclusion

This laboratory demonstrates the fundamental building blocks of a Kubernetes application environment.

Starting from a local Minikube cluster, the project covers:

```text
Cluster
  ↓
Namespaces
  ↓
Configuration & Secrets
  ↓
Pods
  ↓
Services
  ↓
Deployments
  ↓
Ingress
  ↓
Persistent Storage
  ↓
StatefulSets
  ↓
DaemonSets
  ↓
Headlamp
```

The main goal is to understand not only how to create individual Kubernetes resources, but also **how they interact with each other to form a complete application platform**.

This knowledge provides a foundation for working with larger Kubernetes environments, including managed platforms such as Amazon EKS, Azure AKS, and Google Kubernetes Engine.
