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

Helm can also package internal applications.

Example Organization:

```text
best-commerce
```

Example Microservices:

* payments
* shipping

Each microservice can have its own Helm Chart.

---

# Creating a Helm Chart

## Create Chart Structure

```bash
helm create payments
helm create shipping
```

Generated Structure:

```text
payments/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
```

---

# Understanding Chart Structure

## Chart.yaml

Stores chart metadata.

Example:

```yaml
apiVersion: v2
name: payments
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

Contains Kubernetes manifests.

Examples:

* Deployment
* Service
* ConfigMap
* Secret
* ServiceAccount
* Ingress

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

Used to customize templates.

Common configurations:

* Replica count
* Image name
* Image tag
* Resource limits
* Environment variables
* Service type

Example:

```yaml
image:
  repository: busybox
  tag: latest
  pullPolicy: IfNotPresent

appMessage: "I am payment service"
```

---

# Example Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
```

---

# Template Variables

Helm templates use placeholders.

Examples:

| Variable                         | Purpose                |
| -------------------------------- | ---------------------- |
| `{{ .Values.image.repository }}` | Reads from values.yaml |
| `{{ .Release.Name }}`            | Current release name   |
| `{{ .Chart.Name }}`              | Chart name             |

---

# Packaging Helm Charts

## Package Payments Chart

```bash
helm package payments
```

## Package Shipping Chart

```bash
helm package shipping
```

Generated:

```text
payments-0.1.0.tgz
shipping-0.1.0.tgz
```

---

# Creating Helm Repository Index

```bash
helm repo index .
```

Generated:

```text
index.yaml
```

This file contains repository metadata and chart references.

---

# Hosting Internal Helm Repositories

Internal repositories can be hosted on:

* Nexus
* JFrog Artifactory
* GitHub Pages
* S3 Buckets
* Internal Web Servers

---

# Using Internal Repositories

## Add Repository

```bash
helm repo add best-commerce https://example.com/charts
```

## Search Charts

```bash
helm search repo best-commerce
```

## Install Payments Service

```bash
helm install payments best-commerce/payments
```

---

# Helm Upgrade

Upgrade application versions or configurations.

```bash
helm upgrade payments best-commerce/payments
```

Example with custom values:

```bash
helm upgrade payments best-commerce/payments \
  --set replicaCount=3
```

---

# Passing Custom Values

Values can be overridden during installation.

## Using --set

```bash
helm install payments best-commerce/payments \
  --set image.tag=v2
```

## Using Custom Values File

```bash
helm install payments best-commerce/payments \
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

Different values files can be maintained:

```text
values-dev.yaml
values-uat.yaml
values-prod.yaml
```

---

# Common Helm Commands

| Command            | Purpose                  |
| ------------------ | ------------------------ |
| `helm version`     | Verify Helm installation |
| `helm repo add`    | Add repository           |
| `helm repo update` | Update repositories      |
| `helm search repo` | Search charts            |
| `helm install`     | Install chart            |
| `helm list`        | List releases            |
| `helm uninstall`   | Remove release           |
| `helm upgrade`     | Upgrade release          |
| `helm show values` | Show configurable values |
| `helm package`     | Package chart            |
| `helm create`      | Generate chart template  |

---

# Helm Best Practices

## Recommended Practices

* Use separate values files per environment
* Keep templates reusable
* Store charts in centralized repositories
* Use semantic versioning
* Avoid hardcoding values
* Keep charts modular
* Use CI/CD pipelines for chart releases

---

# Real-World DevOps Usage

Helm is heavily used for:

* Platform engineering
* Kubernetes automation
* GitOps workflows
* CI/CD pipelines
* Internal platform tooling
* Multi-environment deployments
* Microservices packaging

Popular tools installed using Helm:

* Prometheus
* Grafana
* Argo CD
* Istio
* NGINX Ingress
* Cert Manager
* External DNS
* AWS Load Balancer Controller

---

# Helm Workflow Summary

```text
1. Add Repository
2. Search Chart
3. Install Chart
4. Customize Values
5. Upgrade Releases
6. Uninstall When Needed
7. Create Internal Charts
8. Publish Charts to Repository
```

---

# Key Takeaways

* Helm is the package manager for Kubernetes
* Charts package Kubernetes applications
* Repositories store Helm Charts
* Releases are deployed chart instances
* values.yaml customizes deployments
* templates/ stores Kubernetes manifests
* Helm simplifies Kubernetes application lifecycle management
* Internal applications can also be packaged and shared using Helm

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
