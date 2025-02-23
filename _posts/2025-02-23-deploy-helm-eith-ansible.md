---
title: "Deploying Helm Charts with Ansible"
date: 2025-02-23
categories:
  - Blog
tags:
  - Ansible
  - Helm
---

### Automating Helm Deployments with Ansible

Helm is a powerful tool for managing Kubernetes applications, but deploying charts manually can become tedious and error-prone. By leveraging Ansible, you can automate Helm deployments, ensuring consistency and reducing operational overhead.

In this post, I'll walk through an approach to deploying Helm charts using Ansible and share my [Ansible role for Helm](https://github.com/dzatulin/ansible/tree/master/roles/ansible-role-helm).

---

### Helm Deployment Using Ansible

The following Ansible playbook deploys **ArgoCD** to an EKS cluster using Helm:

```yaml
- name: Deploy ArgoCD
  hosts: eks_clusters
  vars:
    helm_release_name: argocd
    helm_namespace: argocd
    helm_chart_url: argocd/argo-cd
    helm_repo_url: https://argoproj.github.io/argo-helm
    helm_values_file: ../values/argocd/{{ env }}-values.yaml
    helm_chart_version: 7.3.11
    diff: false
  roles:
    - helm
```
