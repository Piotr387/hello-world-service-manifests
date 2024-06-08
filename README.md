
### Application deployed on cluster: https://github.com/Piotr387/hello-world-service

### Description:
This repository represents the process of automatically deploying applications to a production cluster. The entire pipeline requires only a commit and push to a remote repository, such as GitHub, from the user.

It also demonstrates that ArgoCD can manage multiple clusters, not just a single one. For this purpose, a cluster named "Dev" was created where the user simply provides the image tag and can easily build an application using GitHub Actions workflow.

ArgoCD can be configured to automatically fetch changes from the repository (with a default interval of 3 minutes). However, in real-world scenarios, we often want changes to be fetched instantly. This is achieved with a webhook that pings ArgoCD to fetch the latest changes from the repository.

A good practice is to use separate repositories: one for the application code and one for the manifests. This allows the repository owner to manage the manifests easily without cluttering the application code repository with unrelated commits.

In section [Commands to set up k8s cluster](#Commands-to-set-up-k8s-cluster) we can see commands which are required to set up whole application. With this approach we can easily add extra cluster, application replicas if needed.


---
### Architecture of CI/CD process:
![](./docs/Chmury%20-%20GitHub%20Actions%20with%20ArgoCD.drawio.svg)

---

### Commands to set up k8s cluster

Project was configured on Argo v2.11.2
https://github.com/argoproj/argo-cd/releases/tag/v2.11.2

```shell
# Verify minikube status:
minikube status

# Start minikube
minikube start

# Create namespace "argocd"
kubectl create namespace argocd

# Install argocd on cluster
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.11.2/manifests/install.yaml

# Command to verify status of starting pods
kubectl -n argocd get pods

# To enter Argo UI we need port foward one of the ports 80/443
kubectl port-forward -n argocd svc/argocd-server 8090:443

# On first launch password we can get from by executing below command
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d 

# Using ArgoCLI command we can easliy configure cluster - instead of clicking on UI
argocd login 127.0.0.1:8090
argocd app list
argocd cluster list

# Create ApplicationSet to kubernetes cluster - which based on configuration will create - namespace, configure application for each namespace and clusters which might have different IP addresses
argocd appset create ./argocd/cluster-addons.yaml

# In case of editing 
argocd appset create ./argocd/cluster-addons.yaml --upsert

# Get IP addresses of services.
minikube service hello-world-service-node-port -n dev --url
minikube service hello-world-service-node-port -n production --url
```