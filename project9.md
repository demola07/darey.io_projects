# **PROJECT 9: TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION.**

### Task

Enhance the architecture prepared in [Project 8](https://github.com/demola07/darey.io_projects/blob/main/project8.md) by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how your updated architecture will look like upon competion of this project:

![project_9_architecture](./project9_images//project_9_architecture.JPG)

**_Project 9 Architecture Diagram_**

---

## INSTALL AND CONFIGURE JENKINS SERVER

### **STEP 1: Install Jenkins server**

1.  Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

    ![jenkins_server](./project9_images//jenkins_server.JPG)

2.  Install JDK (since Jenkins is a Java-based application)

        sudo apt update
        sudo apt install default-jdk-headless

3.  Install Jenkins

         wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
         sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
           /etc/apt/sources.list.d/jenkins.list'
         sudo apt update
         sudo apt-get install jenkins

    Make sure Jenkins is up and running

    ` sudo systemctl status jenkins`

4.  By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

    ![jenkins_port](./project9_images//jenkins_port.JPG)

5.  Perform initial Jenkins setup.
    From your browser access `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`

    **_You will be prompted to provide a default admin password_**

    ![jenkins_page](./project9_images//jenkins_page.JPG)

    ![jenkins_home](./project9_images//jenkins_home.JPG)

### Step 2 – **Configure Jenkins to retrieve source codes from GitHub using Webhooks**

In this part, you will configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings

   ![jenkins_webhook](./project9_images//webhook.JPG)

2. Go to Jenkins web console, click "New Item" and create a "Freestyle project"

   ![jenkins_freestyle_proj](./project9_images//jenkins_freestyle.JPG)

   In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

   ![jenkins_build_1_config](./project9_images//build_1_config.JPG)

   Save the configuration and let us try to run the build. For now we can only do it manually.
   Click "Build Now" button, if you have configured everything correctly, the build will be successfull

   ![jenkins_build_1](./project9_images//jenkins_build_1.JPG)

   **_But this build does not produce anything and it runs only when we trigger it manually. Let us fix it._**

   NB: If you have an error like `stderr: Host key verification failed.`, you can find a solution on [stackoverflow](https://stackoverflow.com/questions/15174194/jenkins-host-key-verification-failed) to resolve the issue

3. Click "Configure" your job/project and add these two configurations

   Configure triggering the job from GitHub webhook:

   ![config_trigger](./project9_images//config_trigger.JPG)

   Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

   ![config_post_build](./project9_images//config_post_build.JPG)

   Make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

   You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

   ![jenkins_webook_success](./project9_images//jenkins_webhook_success.JPG)

   You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

   **_By default, the artifacts are stored on Jenkins server locally_**

   `ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

   ![jenkins_artifacts_no](./project9_images//jenkins_artifacts_dir.JPG)

## CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH

### Step 3: – Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

1.  Install "Publish Over SSH" plugin.

        On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

        On "Available" tab search for "Publish Over SSH" plugin and install it

2.  Configure the job/project to copy artifacts over to NFS server.

        On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

        Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

        1. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
        2. Arbitrary name
        3. Hostname – can be private IP address of your NFS server
        4. Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
        5. Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

    Test the configuration and make sure the connection returns `Success`. Remember, that `TCP port 22` on `NFS` server must be open to receive `SSH` connections.

    ![config_jenkins_to_connect_to_nfs](./project9_images//config_jenkins_to_connect_to_nfs.JPG)

    Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"

    ![post_build_config](./project9_images//post_build_config.JPG)

    ![post_build_config_2](./project9_images//post_build_config_2.JPG)

    Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

    Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

            SSH: Transferred 25 file(s)
            Finished: SUCCESS

    ![success_ssh_connect](./project9_images//success_ssh_connect.JPG)

    **_NB: If there is an error in your build referring to permission issues on the NFS server: you can change the permissions of the `/mnt/apps` directory on the NFS server as follows:_**

            sudo chown -R nobody:nobody /mnt/apps
            sudo chmod -R 777 /mnt/apps

    To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file

    `cat /mnt/apps/README.md`

    If you see the changes you had previously made in your GitHub – the job works as expected.
