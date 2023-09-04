# Project 12 - ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

**In this project you will continue working with `ansible-config-mgt` repository and make some improvements of your code. Now you need to refactor your Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.**

### Step 1 – Jenkins job enhancement

Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require [Copy Artifact](https://plugins.jenkins.io/copyartifact/) plugin

1. Go to your `Jenkins-Ansible` server and create a new directory called `ansible-config-artifact` – we will store there all artifacts after each build.

   ![ansible-config-artifacts_dir](./project12_images//ansible-config-artifacts_dir.JPG)

2. Change permissions to this directory, so Jenkins could save files there –

   `chmod -R 0777 /home/ubuntu/ansible-config-artifact`

   ![ansible-config-artifacts_dir_permission](./project12_images//ansible-config-artifacts_dir_permissions.JPG)

3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

   ![copy_artifact_plugin](./project12_images//copy_artifact_plugin.JPG)

4. Create a new Freestyle project (you have done it in Project 9) and name it `save_artifacts`.

   ![jenkins_config1](./project12_images//jenkins_config1.JPG)

   ![jenkins_config2](./project12_images//jenkins_config2.JPG)

5. This project will be triggered by completion of your existing ansible project. Configure it accordingly:

   **Note: You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your `ansible` job.**

6. The main idea of `save_artifacts` project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory. To achieve this, create a Build step and choose `Copy artifacts from other project`, specify `ansible` as a source project and `/home/ubuntu/ansible-config-artifact` as a target directory.

   ![jenkins_build_artifacts_config](./project12_images///jenkins_build_artifacts.JPG)

7. Test your set up by making some change in README.MD file inside your `ansible-config-mgt` repository (right inside master branch).

   If both Jenkins jobs have completed one after another – you shall see your files inside `/home/ubuntu/ansible-config-artifact` directory and it will be updated with every commit to your master branch.

   Now your Jenkins pipeline is more neat and clean.

   ![jjenkins_build_downstream_copy](./project12_images//jenkins_build_downstream_copy.JPG)

   ![jenkins_build_upstream_copy](./project12_images//jenkins_build_upstream_copy.JPG)

   ![jenkins_server_build_copy](./project12_images//jenkins_server_build_copy.JPG)

### Step 2 – Refactor Ansible code by importing other playbooks into site.yml

**Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.**

![refactor_branch](./project12_images//refactor%20branch.JPG)

Let see code re-use in action by importing other playbooks.

1.  Within `playbooks` folder, create a new file and name it `site.yml` – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, `site.yml` will become a parent to all other playbooks that will be developed. Including `common.yml` that you created previously. Dont worry, you will understand more what this means shortly.

2.  Create a new folder in root of the repository and name it `static-assignments`. The `static-assignments` folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

3.  Move `common.yml` file into the newly created `static-assignments` folder.

4.  Inside `site.yml` file, import `common.yml` playbook.

        ---
        - hosts: all
        - import_playbook: ../static-assignments/common.yml

    The code above uses built in [import_playbook](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_playbook_module.html) Ansible module.

    ![code_new_structure](./project12_images//code_new_structure.JPG)

5.  Run `ansible-playbook` command against the `dev` environment

    Since you need to apply some tasks to your `dev` servers and `wireshark` is already installed – you can go ahead and create another playbook under `static-assignments `and name it `common-del.yml`. In this playbook, configure deletion of `wireshark` utility.

            ---
            - name: update web, nfs and db servers
              hosts: webservers, nfs, db
              remote_user: ec2-user
              become: yes
              become_user: root
              tasks:
              - name: delete wireshark
                yum:
                  name: wireshark
                  state: removed

            - name: update LB server
              hosts: lb
              remote_user: ubuntu
              become: yes
              become_user: root
              tasks:
              - name: delete wireshark
                apt:
                  name: wireshark-qt
                  state: absent
                  autoremove: yes
                  purge: yes
                  autoclean: yes

    ![common_del](./project12_images//common_del.JPG)

    ![jenkins-ansible-server-tree](./project12_images//jenkins-ansible_tree.JPG)

    **_NOTE: You may have to follow the steps below to configure the ansible inventory path_**

    1.  Change the inventory file path in the `/etc/ansible/ansible.cfg` file, to correct folder containing all hosts

    ![ansible_inventory_file_path](./project12_images//inventory_path.JPG)

    2. Test to see if ansible can connect to all hosts

    `ansible all -m ping`

    ![ansible_connection_test](./project12_images//ansible_connection_test.JPG)

    update `site.yml` with - `import_playbook: ../static-assignments/common-del.yml `instead of `common.yml` and run it against `dev` servers:

          cd /home/ubuntu/ansible-config-artifact/

          ansible-playbook -i inventory/dev.yml playbooks/site.yml

    ![ansible_playbook_success](./project12_images//ansible_playbook_success.JPG)

    Make sure that `wireshark` is deleted on all the servers by running `wireshark --version`

    ![nfs_wireshark_removal](./project12_images//nfs_wireshare_removal.JPG)

    ![db_wireshark_removal](./project12_images//db_wireshare_removal.JPG)

    ![web_server_1_wireshark_removal](./project12_images//web_server1_wireshare_removal.JPG)

    ![web_server_2_wireshark_removal](./project12_images//web_server2_wireshare_removal.JPG)

    ![lb_wireshark_removal](./project12_images//lb_wireshare_removal.JPG)

    Now you have learned how to use `import_playbooks` module and you have a ready solution to install/delete packages on multiple servers with just one command.

### Step 3 – Configure UAT Webservers with a role ‘Webserver’

We have our nice and clean `dev` environment, so let us put it aside and configure 2 new Web Servers as `uat`. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.

1.  Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our `uat` servers, so give them names accordingly – `Web1-UAT` and `Web2-UAT`.

    ![uat_servers](./project12_images//uat_servers.JPG)

2.  To create a role, you must create a directory called `roles/`, relative to the playbook file or in `/etc/ansible/` directory.

    There are two ways how you can create this folder structure:

    - Use an Ansible utility called `ansible-galaxy` inside `ansible-config-mgt/role`s directory (you need to create `roles` directory upfront)

             mkdir roles
             cd roles
             ansible-galaxy init webserver

    - Create the directory/files structure manually

      **Note**: You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on `Jenkins-Ansible` server.

      The entire folder structure should look like below, but if you create it manually – you can skip creating `tests`, `files`, and `vars` or remove them if you used `ansible-galaxy`

            └── webserver
            ├── README.md
            ├── defaults
            │ └── main.yml
            ├── files
            ├── handlers
            │ └── main.yml
            ├── meta
            │ └── main.yml
            ├── tasks
            │ └── main.yml
            ├── templates
            ├── tests
            │ ├── inventory
            │ └── test.yml
            └── vars
            └── main.yml

      ![ansible_roles](./project12_images//ansible_roles.JPG)

      After removing unnecessary directories and files, the roles structure should look like this

            └── webserver
            ├── README.md
            ├── defaults
            │ └── main.yml
            ├── handlers
            │ └── main.yml
            ├── meta
            │ └── main.yml
            ├── tasks
            │ └── main.yml
            └── templates

      ![ansible_roles_updated](./project12_images//ansible_roles_updated.JPG)

3.  Update your inventory `ansible-config-mgt/inventory/uat.yml` file with IP addresses of your 2 UAT Web servers

    **NOTE:** Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance just as you have done in [project 11](https://github.com/demola07/darey.io_projects/blob/main/project11.md);

    To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:

    For Windows users – [ssh-agent on windows](https://www.youtube.com/watch?v=OplGrY74qog)

    For Linux users – [ssh-agent on linux](https://www.youtube.com/watch?v=RRRQLgAfcJw)

            [uat-webservers]
            <Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

            <Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2

    ![uat_file](./project12_images//uat_file.JPG)

4.  In `/etc/ansible/ansible.cfg` file uncomment `roles_path` string and provide a full path to your roles directory `roles_path = /home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles.

    ![ansible_role_config_path](./project12_images//ansible_roles_config_path.JPG)

5.  It is time to start adding some logic to the webserver role. Go into `tasks` directory, and within the `main.yml` file, start writing configuration tasks to do the following:

    - Install and configure Apache (`httpd` service)
    - Clone Tooling website from GitHub `https://github.com/<your-name>/tooling.git`.
    - Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
      = Make sure `httpd` service is started

      Your `main.yml` may consist of following tasks:

            ---
            - name: install apache
            become: true
            ansible.builtin.yum:
               name: "httpd"
               state: present

            - name: install git
            become: true
            ansible.builtin.yum:
               name: "git"
               state: present

            - name: clone a repo
            become: true
            ansible.builtin.git:
               repo: https://github.com/<your-name>/tooling.git
               dest: /var/www/html
               force: yes

            - name: copy html content to one level up
            become: true
            command: cp -r /var/www/html/html/ /var/www/

            - name: Start service httpd, if not started
            become: true
            ansible.builtin.service:
               name: httpd
               state: started

            - name: recursively remove /var/www/html/html/ directory
            become: true
            ansible.builtin.file:
               path: /var/www/html/html
               state: absent

      ![ansible_task_main.yml](./project12_images//ansible_task_main.yml.JPG)

### Step 4 – Reference ‘Webserver’ role

Within the `static-assignments` folder, create a new assignment for **uat-webservers** `uat-webservers.yml`. This is where you will reference the role.

      ---
      - hosts: uat-webservers
      roles:
         - webserver

![uat_webserver_assignment](./project12_images//uat_webserver_assignment.JPG)

Remember that the entry point to our ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml`.

So, we should have this in `site.yml`

      ---
      - hosts: all
      - import_playbook: ../static-assignments/common.yml

      - hosts: uat-webservers
      - import_playbook: ../static-assignments/uat-webservers.yml

![site.yml_file](./project12_images//site.yml.JPG)

### Step 5 – Commit & Test

Commit your changes, create a Pull Request and merge them to `master / main` branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your `Jenkins-Ansible` server into `/home/ubuntu/ansible-config-mgt/ & /home/ubuntu/ansible-config-artifact/` directory.

![jenkins_builds](./project12_images//jenkins_builds.JPG)

Now run the playbook against your `uat` inventory and see what happens:

      ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml

![uat_playbook_success1](./project12_images//uat_playbook_success1.JPG)
![uat_playbook_success2](./project12_images//uat_playbook_success2.JPG)

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

**_NOTE: Do not forget to open `http port 80` on 2 fresh EC2 instances_**

`http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php`

![web1](./project12_images//web1.JPG)

or

`http://<Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php`

![web2](./project12_images//web2.JPG)

Your Ansible architecture now looks like this:

![final_architecture](./project12_images//final_architecture.JPG)

### Code containing all configuration can be found in the [ansible-config-mgt](https://github.com/demola07/ansible-config-mgt) repository
