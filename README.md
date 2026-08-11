Project 12: Ansible Refactoring & Static Assignments


1. Configure Jenkins Save Artifacts Job

   1-Log into your Jenkins-Ansible control server terminal and create the artifacts directory:

      sudo mkdir -p /home/ubuntu/ansible-config-artifact

      sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact
   
   2- Open the Jenkins Web Console, navigate to Manage Jenkins $\rightarrow$ Plugins, search for Copy Artifact, and install it without restarting Jenkins.

   3- Create a new Freestyle Project named save_artifacts.

   4- Under General, select Discard old builds and set Max # of builds to keep to 2.

   5- Under Build Triggers, select Build after other projects are built, enter ansible in Projects to watch, and choose Trigger only if build is stable.

   6- Under Build Steps, add Copy artifacts from another project:

          -Project name: ansible

          -Target directory: /home/ubuntu/ansible-config-artifact

          -Artifacts to copy: **

    7-Click Save.



2- Refactor Repository & Create Playbook Imports

  1- Navigate to your local project directory and pull the latest code from the main branch:
     cd ~/ansible-config-mgt
   
     git checkout master
     
     git pull
     
     git checkout -b refactor

  2- Create the static-assignments folder and relocate your common tasks:
     
     mkdir -p static-assignments
     
     mv playbooks/common.yml static-assignments/
  
  3- Create a cleanup playbook to safely handle package removal across hosts:
     
     cat << 'EOF' > static-assignments/common-del.yml
     ---
     - name: remove wireshark from redhat hosts
       hosts: webservers, nfs, db
       remote_user: ec2-user
       become: yes
       tasks:
     - name: remove wireshark package
        yum:
        name: wireshark
        state: absent
        ignore_errors: yes

      - name: remove wireshark from ubuntu hosts
        hosts: lb
        remote_user: ubuntu
        become: yes
        tasks:
           - name: remove wireshark package
            apt:
            name: wireshark
            state: absent
            autoremove: yes
            ignore_errors: yes
    EOF

  
  4- Create the main entry point playbook:
     
     
     cat << 'EOF' > playbooks/site.yml
     ---
     - hosts: all
     - import_playbook: ../static-assignments/common-del.yml
     - import_playbook: ../static-assignments/uat-webservers.yml
     EOF


3. Initialize & Build the Webserver Role


   1- Create the roles directory and initialize the webserver role structure:

      mkdir -p roles

      cd roles

      ansible-galaxy init webserver

      cd ..

   2- Update your inventory file to target your UAT nodes (172.31.8.200 and 172.31.0.22):


      cat << 'EOF' >
       inventory/uat.yml

       [uat-webservers]

        172.31.8.200 ansible_ssh_user='ec2-user'

        172.31.0.22 ansible_ssh_user='ec2-user'

       EOF

   3- Update /etc/ansible/ansible.cfg to ensure Ansible locates your roles path:

     sudo sed -i 's|^#roles_path.*|roles_path = /home/ubuntu/ansible-config-mgt/roles|' /etc/ansible/ansible.cfg

   4- Define deployment tasks inside the webserver role:


       cat << 'EOF' > roles/webserver/tasks/main.yml
      ---
      - name: install apache web server
         become: true
         ansible.builtin.yum:
         name: "httpd"
         state: present

     - name: install git
        become: true
        ansible.builtin.yum:
        name: "git"
        state: present

     - name: clone tooling repository
        become: true
        ansible.builtin.git:
        repo: "https://github.com/your-github-username/tooling.git"
        dest: "/var/www/html"
        force: yes

    - name: copy internal application assets to web root
       become: true
       command: cp -r /var/www/html/html/ /var/www/
       ignore_errors: yes

     - name: start and enable httpd service
        become: true
        ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes

     - name: remove temporary nested asset directory
        become: true
        ansible.builtin.file:
       path: /var/www/html/html
       state: absent
      EOF

  
  5- Map the role to your static assignment mapping file:
      
         cat << 'EOF' > static-assignments/uat-webservers.yml
         ---
         - hosts: uat-webservers
           roles:
         - webserver
         EOF


4. Commit Changes, Execute Deployment & Workarounds


   1- Commit all additions and push your branch to GitHub:
         git add .

         git commit -m "feat: complete ansible refactoring, static assignments and webserver role"

         git push origin refactor

   2- Merge your Pull Request into master via GitHub. Confirm that Jenkins triggers and copies artifacts automatically.


   3- Load your SSH key into your SSH agent on your local machine / control node:


      eval $(ssh-agent -s)

      ssh-add /path/to/your-key.pem


   4- Execute the playbook execution against the UAT environment:

        cd /home/ubuntu/ansible-config-mgt

        ANSIBLE_ROLES_PATH=./roles ansible-playbook -i inventory/uat.yml playbooks/site.yml


   5-  Verify deployment success by visiting the public IP addresses of UAT1 and UAT2 in your browser:
 
       http://<UAT1-PUBLIC-IP>/index.php

       http://<UAT2-PUBLIC-IP>/index.php
     
