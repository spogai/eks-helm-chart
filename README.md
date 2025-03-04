# SPOG Agent Installation Guide

## Overview
This document provides step-by-step instructions for installing the SPOG Agent in an Amazon EKS (Elastic Kubernetes Service) cluster. You can choose to install the agent using Helm or directly without Helm by leveraging the GitHub URL.

---

## Prerequisites

- An active AWS account with permissions to manage EKS clusters.
- An EKS cluster is already set up and configured.
- Helm 3.8 or above installed on your local machine and initialized with the Kubernetes cluster (if using Helm).
- `kubectl` is installed and configured to interact with the EKS cluster.
- Base64 Encoded Webhook URL Endpoint created by adding an EKS Integration in SPOG.

---

## Installing SPOG Agent Using Helm

### Step 1: Add SPOG Helm Repository
Run the following commands to add the SPOG Helm repository and update it:

```sh
helm repo add spog-eks https://spogai.github.io/eks-helm-chart
helm repo update
```

### Step 2: Install SPOG Agent
Install the `spog-eks-agent` Helm chart with customized values using the `--set` option:

```sh
helm install spog-eks-agent spog-eks/spog-eks \
  --set cronjob.schedule="* */23 * * *" \
  --set cronjob.image.repository="spogai/eks" \
  --set cronjob.image.tag="v2.30" \
  --set cronjob.image.pullPolicy="IfNotPresent" \
  --set cronjob.restartPolicy="Never" \
  --set Namespace="spog" \
  --set Clustername="<CLUSTERNAME>" \
  --set spog_host_url="https://<SPOG_INSTANCE_NAME>.spog.ai" \
  --set spog_api_username="<USERNAME>" \
  --set spog_api_password="<PASSWORD_IN_BASE64ENCODED>" \
  --set spog_integration_id="<KUBERNETES_INTEGRATION_ID>" \
  --set cluster_master_ip_or_arn="<MASTERIP_K8S_CLUSTER_OR_AWS_ARN>" \
  --set k8sinspect_integration_id="<KUBERNETES_INTEGRATION_ID>"
```

**Note:** Replace placeholders like `<CLUSTERNAME>` and `<PASSWORD_IN_BASE64ENCODED>` with actual values. You can seek assistance from the SPOG SPOC if needed.

If an existing `spog-eks-agent` release is present, uninstall it to avoid conflicts:

```sh
helm uninstall spog-eks-agent
```

### Step 3: Verify Installation
Run the following command to verify the installation:

```sh
kubectl get all -n spog
```

Expected output:

```sh
NAME                                   READY   STATUS      RESTARTS   AGE
pod/spog-eks-agent-cj-28634992-rqz8t   0/1     Completed   0          2m17s
pod/spog-eks-agent-cj-28634994-pd99b   0/1     Completed   0          17s

NAME                              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/spog-eks-agent-cj   * */23 * * *   False     0        17s             3m36s

NAME                                   COMPLETIONS   DURATION   AGE
job.batch/spog-eks-agent-cj-28634992   1/1           27s        2m17s
job.batch/spog-eks-agent-cj-28634994   1/1           9s         17s
```

---

## Installing SPOG Agent Without Helm

### Step 1: Download Chart Package
Download the Helm chart package directly from the GitHub URL:

```sh
wget https://github.com/spogai/eks-helm-chart/blob/gh-pages/spog-eks-0.1.12.tgz
```

### Step 2: Extract Package
Extract the downloaded `.tgz` file:

```sh
tar -xvf spog-eks-0.1.12.tgz
```

### Step 3: Customize Values
Navigate to the extracted directory and open the `values.yaml` file. Modify it to include the necessary configuration values:

```yaml
cronjob:
  schedule: "* */23 * * *"
  image:
    repository: "spogai/eks"
    tag: "v2.30"
    pullPolicy: "IfNotPresent"
  restartPolicy: "Never"
Namespace: "spog"
Clustername: "<CLUSTERNAME>"
spog_host_url: "https://<SPOG_INSTANCE_NAME>.spog.ai"
spog_api_username: "<USERNAME>"
spog_api_password: "<PASSWORD_IN_BASE64ENCODED>"
spog_integration_id: "<KUBERNETES_INTEGRATION_ID>"
cluster_master_ip_or_arn: "<MASTERIP_K8S_CLUSTER_OR_AWS_ARN>"
k8sinspect_integration_id: "<KUBERNETES_INTEGRATION_ID>"
```

### Step 4: Apply Manifest Files
Apply the manifest files included in the extracted directory to deploy the SPOG Agent:

```sh
kubectl apply -f ./manifests/
```

### Step 5: Verify Installation
Check the status of the deployed resources:

```sh
kubectl get all -n spog
```

---

## Troubleshooting

### Q1: How do I check if Helm is installed properly on my machine?
You can install Helm by following the [official Helm documentation](https://helm.sh/docs/intro/install/). To validate whether Helm is installed properly, execute the following command:

```sh
helm version --short
```

### Q2: How do I verify that the Helm chart for SPOG has been installed correctly?
You can search for available Helm charts in the `spog-eks` repository:

```sh
helm search repo spog-eks
```

### Q3: How do I install and configure `kubectl` to work with my EKS cluster?
Follow the [official Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to install `kubectl` and configure it for your EKS cluster.

---

## Summary of Data Collected Using SPOG Agent

The data collected during the installation and monitoring of the SPOG Agent in an EKS cluster includes:

- **Cluster Information:** Details of the cluster such as name, namespace, and master node IP.
- **Service and Pod Details:** Information about running services, pods, and their statuses.
- **Ingress Configuration:** Host rules, paths, and service-port mappings for ingress controllers.
- **Security and Compliance Reports:** Kubernetes CIS benchmark reports identifying vulnerabilities and security risks, categorized as CRITICAL, HIGH, MEDIUM, and LOW severity.
- **Log and Metrics Data:** Collected through Fluent Bit and other monitoring tools.
- **Bogus or Misconfigured Services:** Identification of misconfigured services that need attention.

This data helps in identifying potential security risks, monitoring cluster health, and ensuring compliance with security best practices.
