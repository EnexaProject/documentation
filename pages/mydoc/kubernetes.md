---
title: Kubernetes Deployment Guide
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: kubernetes.html           
folder: mydoc
---
# Kubernetes Deployment Guide

## Overview
For development purposes, you can use **Minikube** to create a single-node Kubernetes cluster on your local machine. However, for production or realistic testing environments, we strongly recommend setting up a multi-node cluster using **kubeadm**.

This guide will help you install Kubernetes version **1.31** using kubeadm. Refer to the official tutorial and an additional detailed guide linked below.

## Recommended Installation Guides
1. **Official Kubernetes Installation Tutorial**: [Install kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
2. **Comprehensive Blog Tutorial**: [Install Kubernetes with kubeadm](https://docs.dman.cloud/posts/install-kubernetes-with-kubeadm/)

Follow the instructions in these guides to successfully set up your Kubernetes cluster.

## Post-Installation Steps
After installing the cluster, apply all the necessary YAML files for your configuration. Ensure you replace `[url]` with the actual URL containing the YAML files.

```sh
kubectl apply -f [yaml file]
```

## Notes
- Use kubeadm version 1.31 to ensure compatibility.
- Test your cluster with a sample application deployment to verify everything works correctly.
- Check the cluster health using:
    ```sh
    kubectl get nodes
    kubectl get pods -A
    ```

By following these steps and guides, you'll have a fully functional Kubernetes cluster ready for deployment.