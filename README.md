# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.

# 🚀 End-to-End Azure DevOps GitOps Pipeline with AKS & ArgoCD

This project demonstrates a production-grade CI/CD workflow built on GitOps practices, using Azure DevOps Pipelines, Azure Kubernetes Service (AKS), Azure Container Registry (ACR), and ArgoCD. It automates everything from Docker image builds to Kubernetes deployments.

---

## 📚 Contents
- [Introduction](#-1-introduction)
- [Architecture Summary](#-2-architecture-summary)
- [Technology Overview](#-3-technology-overview)
- [Infrastructure Provisioning](#-4-infrastructure-provisioning)
- [Azure DevOps Pipeline Setup](#-5-azure-devops-pipeline-setup)
- [GitOps with ArgoCD](#-6-gitops-with-argocd)
- [Deployment Workflow](#-7-deployment-workflow)
- [Application Access](#-8-accessing-the-services)
- [Highlights](#-9-key-highlights)

---

## 🔰 1. Introduction

This repository contains a complete CI/CD solution for a microservices-based voting application. The goal is to ensure that:

- Code changes automatically build new Docker images  
- Updated images are pushed to ACR  
- Kubernetes manifests are patched with the latest tags  
- ArgoCD detects changes and syncs them into the AKS cluster  

This follows the GitOps philosophy, where Git serves as the source of truth for Kubernetes state.

The application is derived from the classic **Docker Voting App**.

---

## 🧱 2. Architecture Summary

**High-level CI/CD flow:**

1. Developer pushes code → Azure DevOps pipeline triggers  
2. Pipeline builds Docker image  
3. Image is pushed to Azure Container Registry  
4. Kubernetes manifests are updated with the new image tag  
5. ArgoCD automatically syncs the cluster with Git  

---

## 🛠️ 3. Technology Overview

| Category                 | Tool/Service                         |
|-------------------------|---------------------------------------|
| Cloud Platform          | Microsoft Azure                       |
| Kubernetes              | Azure Kubernetes Service (AKS)        |
| Container Registry      | Azure Container Registry (ACR)        |
| CI/CD Engine            | Azure DevOps Pipelines                |
| GitOps Controller       | ArgoCD                                |
| Container Runtime       | Docker                                |
| Databases Used          | PostgreSQL, Redis                     |

---

## ☁️ 4. Infrastructure Provisioning

### **1. Create Resource Group**
```bash
az group create --name votingapp-rg --location eastus
```

### **2. Create ACR**
```bash
az acr create \
  --resource-group votingapp-rg \
  --name VotingAppRegistry \
  --sku Basic
```

### **3. Deploy AKS Cluster**
```bash
az aks create \
  --resource-group votingapp-rg \
  --name azuredevops \
  --node-count 2 \
  --enable-addons monitoring \
  --generate-ssh-keys
```

### **4. Attach ACR to AKS**
```bash
az aks update \
  --resource-group votingapp-rg \
  --name azuredevops \
  --attach-acr VotingAppRegistry
```

### **5. Configure kubectl Context**
```bash
az aks get-credentials \
  --resource-group votingapp-rg \
  --name azuredevops \
  --overwrite-existing
```

Verify:
```bash
kubectl get nodes
```

### **6. Open NodePort Access (NSG Rules)**

Get NSG:
```bash
NSG_NAME=$(az network nsg list --resource-group MC_votingapp-rg_azuredevops_eastus --query "[0].name" -o tsv)
```
Allow ports:

# Vote service - 31000
az network nsg rule create \
  --resource-group MC_votingapp-rg_azuredevops_eastus \
  --nsg-name $NSG_NAME \
  --name AllowVoteNodePort \
  --priority 100 \
  --destination-port-ranges 31000 \
  --access Allow \
  --protocol Tcp

# Result service - 31001
az network nsg rule create \
  --resource-group MC_votingapp-rg_azuredevops_eastus \
  --nsg-name $NSG_NAME \
  --name AllowResultNodePort \
  --priority 101 \
  --destination-port-ranges 31001 \
  --access Allow \
  --protocol Tcp
