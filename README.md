Project 12: Ansible Refactoring & Static Assignments


# Ansible Refactoring & Static Assignments (Imports & Roles)

## Architecture Overview
This repository contains automated infrastructure configuration scripts using Ansible static assignments and reusable roles.

* **Control Node**: Jenkins / Ansible Master (Ubuntu)
* **Target Nodes**: 
  * `UAT Webserver 1`: `172.31.8.200` (RHEL 8)
  * `UAT Webserver 2`: `172.31.0.22` (RHEL 8)

## Automation Workflow
1. Git commit triggers Webhook to primary Jenkins job.
2. Secondary Jenkins job `save_artifacts` copies assets to persistent storage (`/home/ubuntu/ansible-config-artifact`).
3. Central playbook `playbooks/site.yml` imports static assignments and triggers the `webserver` role.
4. Target nodes install Apache, clone source code, and serve application endpoints.

## Execution Command
```bash
ansible-playbook -i inventory/uat.yml playbooks/site.yml
