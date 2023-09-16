# Project 14 - CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP

## Project Description:

![project14_architecture](./project14_images//project_14_architecture_diagram.JPG)

In this project, I will be setting up a CI/CD Pipeline for a PHP based application. The overall CI/CD process looks like the architecture above.

This project is architected in two major repositories with each repository containing its own CI/CD pipeline written in a Jenkinsfile.

- ansible-config-mgt REPO: this repository contains JenkinsFile which is responsible for setting up and configuring infrastructure required to carry out processes required for our application to run. It does this through the use of ansible roles. This repo is infrastructure specific

- PHP-todo REPO : this repository contains jenkinsfile which is focused on processes which are application build specific such as building, linting, static code analysis, push to artifact repository etc


## Prerequisites

Will be making use of AWS virtual machines for this and will require 6 servers for the project which includes: Nginx Server: This would act as the reverse proxy server to our site and tool.

**Jenkins server:** To be used to implement your CI/CD workflows or pipelines. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 8080

**SonarQube server:** To be used for Code quality analysis. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 9000

**Artifactory server:** To be used as the binary repository where the outcome of your build process is stored. Select a t2.medium at least and Security group should be open to port 8081

**Database server:** To server as the databse server for the Todo application

**Todo webserver:** To host the Todo web application.

## Environments

    ├── ci
    ├── dev
    ├── pentest
    ├── pre-prod
    ├── prod
    ├── sit
    └── uat


**ci inventory file**

    [jenkins]
    <Jenkins-Private-IP-Address>

    [nginx]
    <Nginx-Private-IP-Address>

    [sonarqube]
    <SonarQube-Private-IP-Address>

    [artifact_repository]
    <Artifact_repository-Private-IP-Address>

**dev Inventory file**

    [tooling]
    <Tooling-Web-Server-Private-IP-Address>

    [todo]
    <Todo-Web-Server-Private-IP-Address>

    [nginx]
    <Nginx-Private-IP-Address>

    [db:vars]
    ansible_user=ec2-user
    ansible_python_interpreter=/usr/bin/python

    [db]
    <DB-Server-Private-IP-Address>

**pe  ntest inventory file**

    [pentest:children]
    pentest-todo
    pentest-tooling

    [pentest-todo]
    <Pentest-for-Todo-Private-IP-Address>

    [pentest-tooling]
    <Pentest-for-Tooling-Private-IP-Address>

 ### ANSIBLE ROLES FOR CI ENVIRONMENT
 ---

 To automate the setup of `SonarQube` and `JFROG Artifactory`, we can use `ansible-galaxy` to install this configuration into our ansible roles which will be used and run against the `sonarqube server` and `artifactory server`.

We will see this in play later


### Configuring Ansible For Jenkins Deployment
---

1. Navigate to Jenkins URL

2. Install & Open Blue Ocean Jenkins Plugin

3. Create a new pipeline

    ![blue_ocean](./project14_images//blue_ocean.JPG)

4. Select GitHub 

    ![blue_ocean](./project14_images//blue_ocean1.JPG)

5. Connect Jenkins with GitHub

    ![blue_ocean](./project14_images//blue_ocean2.JPG)

6. Login to GitHub & Generate an Access _`settings => Developer Settings => OAuth Apps => Tokens`_

    ![github tokens](./project14_images//github_tokens.JPG)

7. Copy Access Token

    ![github tokens](./project14_images//github_tokens1.JPG)

8. Paste the token and connect

9. Create a new Pipeline

    ![github tokens](./project14_images//jenkins_pipeline.JPG)


At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.

   ![blue ocean](./project14_images//blue_ocean3.JPG)

Here is our newly created pipeline. It takes the name of your GitHub repository.

   ![jenkins pipeline](./project14_images//jenkins_pipeline1.JPG)


### Let us create our Jenkinsfile

Inside the Ansible project, create a new directory `deploy` and start a new file `Jenkinsfile` inside the directory.


Add the code snippet below to start building the `Jenkinsfile` gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the `shell script` module to echo `Building Stage`

![Jenkinsfile](./project14_images//Jenkinsfile.JPG)

Now go back into the Ansible pipeline in Jenkins, and select configure

![ansible_proj](./project14_images//ansible_jen.JPG)

Scroll down to `Build Configuration` section and specify the location of the Jenkinsfile at `deploy/Jenkinsfile`

![ansible_proj](./project14_images//ansible_jen1.JPG)

Back to the pipeline again, this time click "Build now"

![jenkins build](./project14_images//jenkins_build.JPG)

This will trigger a build and you will be able to see the effect of our basic `Jenkinsfile` configuration by going through the console output of the build.

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

1. Click on Blue Ocean

2. Select your project

3. Click on the button against the branch

![jenkins build](./project14_images//jenkins_build1.JPG)

![jenkins build](./project14_images//jenkins_build2.JPG)

![jenkins build](./project14_images//jenkins_build3.JPG)

Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

Let us see this in action.

1. Create a new git branch and name it `feature/jenkinspipeline-stages`

2. Currently we only have the Build stage. Let us add another stage called `Test`. Paste the code snippet below and push the new changes to GitHub.

![jenkins branch](./project14_images//jenkins_branch.JPG)

3. To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

    i.  Click on the "Administration" button
    ![jenkins multibranch](./project14_images//jenkins_multibranch.JPG)

    ii. Navigate to the Ansible project and click on "Scan repository now"

    iii. Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

    ![jenkins multibranch](./project14_images//jenkins_multibranch1.JPG)

    iv. In Blue Ocean, you can now see how the `Jenkinsfile` has caused a new step in the pipeline launch build for the new branch.

    ![jenkins multibranch](./project14_images//jenkins_multi_blue_ocean.JPG)

    ![jenkins multibranch](./project14_images//jenkins_multi_blue_ocean1.JPG)

    
    
## RUNNING ANSIBLE PLAYBOOK FROM JENKINS

Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

1. Installing Ansible on Jenkins

2. Installing Ansible plugin in Jenkins UI

3. Creating `Jenkinsfile` from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully).

    You can watch a [10 minutes video here](https://www.youtube.com/watch?v=PRpEbFZi7nI&feature=youtu.be) to guide you through the entire setup

    **Note:** Ensure that Ansible runs against the Dev environment successfully.