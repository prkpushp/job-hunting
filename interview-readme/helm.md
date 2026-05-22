# Helm - Complete Guide for DevOps & Kubernetes

## Overview

Helm is a package manager for Kubernetes that simplifies the installation, upgrade, rollback, and management of Kubernetes applications.

Just like `apt` for Ubuntu or `yum` for RHEL-based systems, Helm helps manage Kubernetes packages called **Charts**.

Using Helm, DevOps engineers can:

* Install Kubernetes applications quickly
* Manage versions of applications
* Upgrade or rollback deployments
* Uninstall applications cleanly
* Package internal applications as reusable Helm Charts
* Maintain reusable and customizable deployments across environments

---

# Why Helm?

Without Helm, installing applications on Kubernetes usually requires:

* Multiple YAML manifests
* Manual dependency handling
* Separate scripts for different environments
* Continuous maintenance of deployment files

For example, installing applications like:

* Prometheus
* Grafana
* Argo CD
* NGINX Ingress Controller
* AWS Load Balancer Controller

may require:

* Deployments
* Services
* ConfigMaps
* ServiceAccounts
* RBAC
* StatefulSets
* Secrets
* Ingress resources

Managing all these manually becomes difficult at scale.

Helm solves this problem by packaging everything into reusable charts.

---

# What Helm Can Do

Helm supports:

| Feature   | Description                                   |
| --------- | --------------------------------------------- |
| Install   | Deploy applications/controllers on Kubernetes |
| Upgrade   | Upgrade application versions                  |
| Rollback  | Revert to previous versions                   |
| Uninstall | Remove applications cleanly                   |
| Package   | Bundle Kubernetes manifests into Helm Charts  |
| Share     | Publish charts for teams or organizations     |

---

# Helm Core Components

## 1. Chart

A **Chart** is a packaged Kubernetes application.

It contains:

* Kubernetes YAML manifests
* Templates
* Metadata
* Configuration values

Examples:

* Prometheus Chart
* Grafana Chart
* Argo CD Chart
* NGINX Chart

---

## 2. Repository

A **Repository** stores Helm Charts.

Examples:

* Bitnami
* ArtifactHub
* Internal Nexus/Artifactory repositories

Just like Linux package repositories.

---

## 3. Release

A **Release** is a deployed instance of a Helm Chart.

Example:

```bash
helm install nginx-v1 bitnami/nginx
```

Here:

* `bitnami/nginx` → Chart
* `nginx-v1` → Release Name

---

# Helm Architecture

```text
Helm CLI
   |
   v
Kubeconfig Context
   |
   v
Kubernetes Cluster
```

Helm uses the current Kubernetes context from:

```bash
kubectl config current-context
```

No server-side installation is required in Helm v3.

---

# Installing Helm

## macOS

```bash
brew install helm
```

## Ubuntu/Debian

```bash
sudo apt install helm
```

## Windows (Chocolatey)

```bash
choco install kubernetes-helm
```

## Verify Installation

```bash
helm version
```

---

# Working with Helm Repositories

## Add Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## Update Repositories

```bash
helm repo update
```

## List Repositories

```bash
helm repo list
```

---

# Searching Charts

## Search All Charts

```bash
helm search repo bitnami
```

## Search Specific Chart

```bash
helm search repo bitnami | grep nginx
```

Example charts available:

* nginx
* prometheus
* grafana
* redis
* tomcat
* spark
* thanos

---

# Installing Applications Using Helm

## Install NGINX

```bash
helm install nginx-v1 bitnami/nginx
```

## Verify Deployment

```bash
kubectl get pods
```

---

## Install Prometheus

```bash
helm install prometheus bitnami/prometheus
```

---

# Listing Helm Releases

```bash
helm list
```

Example Output:

```text
NAME         NAMESPACE   REVISION   STATUS
nginx-v1     default     1          deployed
prometheus   default     1          deployed
```

---

# Uninstalling Helm Releases

## Uninstall NGINX

```bash
helm uninstall nginx-v1
```

## Uninstall Prometheus

```bash
helm uninstall prometheus
```

---

# Installing Third-Party Controllers

Not every chart exists in Bitnami.

Example: AWS Load Balancer Controller

## Add EKS Repository

```bash
helm repo add eks https://aws.github.io/eks-charts
```

## Update Repositories

```bash
helm repo update
```

## Search Charts

```bash
helm search repo eks
```

## Install AWS Load Balancer Controller

```bash
helm install alb eks/aws-load-balancer-controller
```

Some charts require additional configuration values.

---

# Viewing Default Values

To inspect configurable values:

```bash
helm show values eks/aws-load-balancer-controller
```

This helps customize:

* Replica count
* Image versions
* Resources
* Service types
* Ingress settings
* Environment-specific configs

---

# Creating Your Own Helm Charts

Helm can also package internal applications used within organizations.

Example Repository:

```text
platform-charts
```

Example Applications:

* frontend-app
* backend-api

Each application can be packaged as an independent Helm Chart.

---

# Creating a Helm Chart

## Generate Chart Structure

```bash
helm create frontend-app
helm create backend-api
```

Generated Structure:

```text
frontend-app/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
```

---

# Understanding Chart Structure

## Chart.yaml

Stores metadata about the chart.

Example:

```yaml
apiVersion: v2
name: frontend-app
version: 0.1.0
appVersion: "1.0.0"
```

Common fields:

* Chart name
* Chart version
* Application version
* Description
* Maintainers

---

## templates/

Contains Kubernetes resource manifests.

Common resources:

* Deployment
* Service
* ConfigMap
* Secret
* Ingress
* ServiceAccount

Example:

```text
templates/
├── deployment.yaml
├── service.yaml
├── ingress.yaml
└── serviceaccount.yaml
```

---

## values.yaml

Used for customizing chart deployments across environments.

Example:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```

Typical customizations:

* Replica count
* Image versions
* Resource limits
* Service types
* Environment variables
* Ingress configuration

---

# Example Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: frontend
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
```

---

# Helm Template Variables

| Variable                         | Description                   |
| -------------------------------- | ----------------------------- |
| `{{ .Values.image.repository }}` | Reads values from values.yaml |
| `{{ .Release.Name }}`            | Current release name          |
| `{{ .Chart.Name }}`              | Helm chart name               |
| `{{ .Values.replicaCount }}`     | Replica configuration         |

---

# Packaging Helm Charts

## Package Frontend Chart

```bash
helm package frontend-app
```

## Package Backend Chart

```bash
helm package backend-api
```

Generated Packages:

```text
frontend-app-0.1.0.tgz
backend-api-0.1.0.tgz
```

---

# Creating Repository Index

```bash
helm repo index .
```

Generated:

```text
index.yaml
```

This file contains metadata about all available charts in the repository.

---

# Hosting Internal Helm Repositories

Organizations commonly host Helm repositories on:

* JFrog Artifactory
* Nexus Repository
* GitHub Pages
* AWS S3
* Internal Web Servers

---

# Using Internal Helm Repositories

## Add Repository

```bash
helm repo add platform-charts https://example.com/charts
```

## Search Available Charts

```bash
helm search repo platform-charts
```

## Install Frontend Application

```bash
helm install frontend platform-charts/frontend-app
```

## Install Backend API

```bash
helm install backend platform-charts/backend-api
```

---

# Helm Upgrade

Upgrade application versions or deployment configurations.

```bash
helm upgrade frontend platform-charts/frontend-app
```

Example with custom values:

```bash
helm upgrade frontend platform-charts/frontend-app \
  --set replicaCount=3
```

---

# Passing Custom Values

## Using --set

```bash
helm install frontend platform-charts/frontend-app \
  --set image.tag=v2
```

## Using Environment-Specific Values Files

```bash
helm install frontend platform-charts/frontend-app \
  -f values-prod.yaml
```

---

# Environment-Specific Deployments

Example:

| Environment | Replicas |
| ----------- | -------- |
| Dev         | 1        |
| UAT         | 2        |
| Production  | 5        |

Common values files:

```text
values-dev.yaml
values-uat.yaml
values-prod.yaml
```

This helps standardize deployments across environments.

---

# Real-World DevOps Usage

Helm is widely used for:

* Kubernetes platform engineering
* CI/CD pipelines
* GitOps workflows
* Multi-environment deployments
* Standardized application packaging
* Infrastructure automation

Popular applications installed using Helm:

* Prometheus
* Grafana
* Argo CD
* Istio
* NGINX Ingress Controller
* Cert Manager
* External DNS
* AWS Load Balancer Controller

Helm enables reusable, versioned, and maintainable Kubernetes deployments.

---

# Helm Workflow Summary

```text
1. Add Repository
2. Search Charts
3. Install Charts
4. Customize Values
5. Upgrade Releases
6. Uninstall Releases
7. Create Internal Charts
8. Publish Charts to Repository
```

---

# Key Takeaways

* Helm is the package manager for Kubernetes
* Charts package Kubernetes applications
* Repositories store reusable Helm Charts
* Releases are deployed chart instances
* values.yaml enables environment-specific customization
* templates/ stores Kubernetes manifests
* Helm simplifies Kubernetes application lifecycle management
* Internal applications can be packaged and shared across teams

---

# Useful References

## Official Helm Documentation

[https://helm.sh/docs/](https://helm.sh/docs/)

## Bitnami Charts

[https://bitnami.com/stacks/helm](https://bitnami.com/stacks/helm)

## Artifact Hub

[https://artifacthub.io/](https://artifacthub.io/)

---

# Conclusion

Helm is one of the most important tools in the Kubernetes ecosystem and is widely used in DevOps, Platform Engineering, and Cloud-native environments.

Understanding:

* Helm repositories
* Charts
* Releases
* Templates
* values.yaml
* Packaging
* Upgrades
* Internal chart development

is essential for modern Kubernetes and DevOps interviews.
