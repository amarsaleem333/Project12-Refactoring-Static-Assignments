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


5. **Commit the Changes**  
   * Scroll down to the **Commit changes...** section.
   * Enter a commit message like: `docs: add README documentation`.
   * Click the green **Commit changes** button.

---

### Method 2: Create a File in VS Code / Local Editor

If you are working inside Visual Studio Code connected to your server:

1. In VS Code's file explorer on the left, right-click inside your root folder `ansible-config-mgt`.
2. Select **New File** and name it `README.md`.
3. Paste the markdown block above directly into the file.
4. Save the file (`Ctrl + S` or `Cmd + S`).
5. Run these two simple commands in your VS Code terminal to upload it:
   ```bash
   git add README.md
   git commit -m "docs: add README file"
   git push origin main


