# Minikube Kubernetes Lab

A beginner-friendly local Kubernetes lab using **Minikube**,
**kubectl**, and **Docker**. This guide covers installation, starting a
cluster, deploying Nginx, and opening the Kubernetes Dashboard.

## Prerequisites

-   Windows 10/11, Ubuntu, or another supported operating system
-   Docker Desktop (Windows/macOS) or Docker Engine (Linux), running
-   Hardware resources appropriate for your workloads
-   Internet access to download container images

> The commands below use PowerShell on Windows unless otherwise noted.
> Run PowerShell as Administrator for installation.

## 1. Install Minikube and kubectl

### Windows (winget)

``` powershell
winget install Kubernetes.minikube
winget install Kubernetes.kubectl
```

Close and reopen PowerShell, then verify:

``` powershell
minikube version
kubectl version --client
docker version
```

### Ubuntu (Minikube)

Follow the official installation instructions for your CPU architecture:
https://minikube.sigs.k8s.io/docs/start/

Install `kubectl` using the official Kubernetes instructions:
https://kubernetes.io/docs/tasks/tools/

## 2. Start the cluster

Ensure Docker is running, then start Minikube with the Docker driver:

``` bash
minikube start --driver=docker --cpus=2 --memory=4096
```

Adjust CPU and memory to suit your machine. The example allocates 2 CPUs
and 4 GB RAM.

## 3. Verify the cluster

``` bash
minikube status
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

The Minikube node should report `Ready`.

## 4. Enable Dashboard and Metrics Server

``` bash
minikube addons enable dashboard
minikube addons enable metrics-server
```

Launch the Dashboard:

``` bash
minikube dashboard
```

Minikube opens the Dashboard URL in your browser. Keep the command
running while you use it.

## 5. Deploy your first application (Nginx)

Create a Deployment:

``` bash
kubectl create deployment nginx --image=nginx
```

Expose it as a NodePort Service:

``` bash
kubectl expose deployment nginx --type=NodePort --port=80
```

Check the resources:

``` bash
kubectl get deployments
kubectl get pods
kubectl get services
```

Open the application:

``` bash
minikube service nginx
```

## 6. Useful commands

  Command                             Description
  ----------------------------------- -----------------------------------------
  `minikube start`                    Start the cluster
  `minikube stop`                     Stop the cluster
  `minikube status`                   Show cluster status
  `minikube ip`                       Print the Minikube node IP
  `minikube dashboard`                Open Kubernetes Dashboard
  `minikube addons list`              List available addons
  `kubectl get nodes`                 List cluster nodes
  `kubectl get pods -A`               List pods in all namespaces
  `kubectl get all -A`                List common resources in all namespaces
  `kubectl delete service nginx`      Delete the Nginx Service
  `kubectl delete deployment nginx`   Delete the Nginx Deployment
  `minikube delete`                   Delete the Minikube cluster

## 7. Troubleshooting

### Docker driver errors

-   Confirm Docker Desktop / Docker Engine is running.
-   Run `docker version` and confirm both client and server information
    appear.
-   Retry with `minikube start --driver=docker`.

### Node is not Ready

``` bash
minikube status
kubectl get nodes
kubectl describe node minikube
```

### Pod is not starting

``` bash
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Replace `<pod-name>` with the actual pod name.

### Restart the cluster

``` bash
minikube stop
minikube start --driver=docker --cpus=2 --memory=4096
```

## 8. Clean up the Nginx demo

``` bash
kubectl delete service nginx
kubectl delete deployment nginx
```

To remove the entire local cluster (this deletes its Kubernetes
resources and local cluster data):

``` bash
minikube delete
```

## Suggested GitHub repository structure

``` text
minikube-kubernetes-lab/
├── README.md
├── manifests/
│   └── nginx/
│       ├── deployment.yaml
│       └── service.yaml
├── screenshots/
└── docs/
    └── troubleshooting.md
```

## Official documentation

-   [Minikube documentation](https://minikube.sigs.k8s.io/docs/)
-   [Kubernetes documentation](https://kubernetes.io/docs/home/)
-   [kubectl command
    reference](https://kubernetes.io/docs/reference/kubectl/)

------------------------------------------------------------------------

**Next lab:** Deploy MongoDB with a PersistentVolumeClaim and expose it
safely inside the Kubernetes cluster.
