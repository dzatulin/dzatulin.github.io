---
title: "Automating Grafana Dashboards Deployment with Ansible"
date: 2025-03-6
categories:
  - Blog
tags:
  - Ansible
  - Grafana
---

Grafana is a powerful monitoring tool, but manually uploading dashboards can be tedious. In this guide, I’ll show how to automate Grafana dashboard deployment using **Ansible**. This approach is useful when managing multiple environments and dynamically adding new dashboards.

## **What will we do?**

We’ll create an Ansible role [`ansible-role-grafana`](https://github.com/dzatulin/ansible/tree/master/roles/ansible-role-grafana) that:  
✅ Creates folders in Grafana if they don’t exist.  
✅ Uploads dashboards to the correct folders.

The role structure looks like this:

```

roles/ansible-role-grafana/
│── tasks/
│   ├── main.yml  # Main playbook tasks
│── templates/
│── vars/
│── dashboards/   # JSON dashboard files
|   |-- Kubernetes
|   |-- Loki
│── README.md

```

We create a playbook file 'deploy_grafana.yml':

```

---
- name: Deploy Grafana Dashboards
  hosts: localhost
  roles:
    - ansible-role-grafana

```

Define environment variables in 'env.yaml':

```

grafana_api: "http://grafana.example.com"
api_token: "your-api-token"
dashboards_dir: "/path/to/dashboards"

```
