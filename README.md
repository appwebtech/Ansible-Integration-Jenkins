# Ansible Integration in Jenkins CI/CD Pipeline

<p float="left">
  <img src="https://github.com/appwebtech/Deploy-Docker-With-Terraform/blob/main/images/docker.png" width="90">

  <img src="https://github.com/appwebtech/Ansible-Automation-App-Deployment/blob/main/images/Ansible-logo.png" width="100">


  <img src="https://github.com/appwebtech/Ansible-Integration-Jenkins/blob/main/images/jenkins.png" width="48">

  <img src="https://github.com/appwebtech/Ansible-Integration-Jenkins/blob/main/images/digital-ocean-logo.png" width="120">

  <img src="https://github.com/appwebtech/Ansible-Integration-Jenkins/blob/main/images/aws-logo.png" width="85">

</p>

----

## Introduction

I will be integrating Ansible in Jenkins automation server to reliably build, test and deploy a Java-Maven application. Instead of installing the tools inside Jenkins server / container, I will use a different approach. I'll create two dedicated servers in Digital Ocean, one for Jenkins and the one for Ansible. This time I'm doing it remotely as it makes sense from a professional setting, also hey, this is Cloud Computing so I'll take advantage of it.

Once Ansible is set up, I'll execute a playbook from Jenkins Pipeline to configure 2 EC2 instances in AWS by installing Docker and Docker-compose on them. It's good to use multi-cloud technologies and when I get well versed with [**Azure**](https://azure.microsoft.com/en-gb/free/cloud-services/), I will do cross-cloud deployments in both AWS and Azure.

The next thing I'll do is create a pipeline in Jenkins and connect a Java Maven application. Then I'll create a Jenkinsfile that executes Ansible Playbook on the remote Ansible server.

## Ansible Node

I will instantiate a droplet and install **ansible** and Python libraries (**boto3** with **botocore**). I'll also verify that Python3 is installed and if not, install it.

I've made the necessary installations and Ansible server is up and running.

![Ansible-DO](./images/image-1.png)

![Ansible-server](./images/image-2.png)

I will also configure AWS credentials to enable connection from Ansible to AWS. I usually have this locally, so I'll create a directory and create the credentials file.

![Ansible-Credentials](./images/image-3.png)

## EC2 Instances

In AWS, I will spin two EC2 instances with t2.micros which will be the **Managed Server Nodes** by the **Ansible Server Node** in Digital Ocean from the Jenkins pipeline.

![AWS-servers](./images/image-4.png)

## Jenkins

I will create a new Jenkins server in Digital Ocean in an Ubuntu (Debian) environment because the other one in AWS has critical projects that I'm working on. I'll install Docker and use it to containerize Jenkins. In a fresh installation, the password is in */var/jenkins_home/secrets/initialAdminPassword* if anyone is using the project. I'll paste mine for demo purposes and because I'll toss the whole server by the time I'll commit and push to GitHub then I'll grab a cuppa as the installations runs the "suggested plugins"..

![Jenkins-1](./images/image-5.png)

![Jenkins-2](./images/image-6.png)

As usual I'll grab a cuppa as the installations runs the "suggested plugins".

![Jenkins-3](./images/image-7.png)

In Jenkins I have installed [SSH Agent](https://plugins.jenkins.io/ssh-agent/) and added SSH key of the Ansible droplet in order to enable connection capabilities. The latest OPENSSH keys are throwing errors in Jenkins, so I converted my key to the old RSA key hash.

```shell
ssh-keygen -p -f ~/.ssh/droplet -m pem -P "" -N ""
```

![Jenkins-4](./images/image-8.png)

I have created a similar key for AWS EC2 instance and added it in Jenkins to enable it connect to AWS EC2 instances just the way I added the one of the droplet to enable connection to Digital Ocean. I have also written the Jenkinsfile configuration to enable Jenkins connect to both Ansible node in DIgital Ocean and AWS EC2 instances.

```Jenkinsfile
pipeline {
    agent any
    stages {
        stage("copy files to ansible server") {
            steps {
                script {
                    echo "copying all files to ansible control node"
                    sshagent(['ansible-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@139.59.167.35:/root"

                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            sh "scp ${keyfile} root@139.59.167.35:/root/ssh-key.pem"
                        }
                    }
                }
            }
        }   
    }
}
```

After pushing to GitHub, I configured the project with Jenkins and created a build which ran successfully. Ansible files have also been copied successfully by Jenkins to Ansible server together with the *ssh-key.pem* file.

![Jenkins-5](./images/image-9.png)

![Jenkins-6](./images/image-10.png)

Console output

```shell
Started by user Joseph
Lightweight checkout support not available, falling back to full checkout.
Checking out git https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git into /var/jenkins_home/workspace/ansible-pipeline@script to read Jenkinsfile
The recommended git tool is: NONE
using credential Github-GCP
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/ansible-pipeline@script/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git # timeout=10
Fetching upstream changes from https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
using GIT_SSH to set credentials ghp credential
 > git fetch --tags --force --progress -- https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/feature/ansible^{commit} # timeout=10
 > git rev-parse feature/ansible^{commit} # timeout=10
Checking out Revision c7eb07cf70f930791fe26da949a502eaf8f42880 (refs/remotes/origin/feature/ansible)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f c7eb07cf70f930791fe26da949a502eaf8f42880 # timeout=10
Commit message: "Fix security warning for pem File"
 > git rev-list --no-walk 99adadbcd93d26f29faf3ef042b77e6a0a70e5a6 # timeout=10
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/ansible-pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
The recommended git tool is: NONE
using credential Github-GCP
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/ansible-pipeline/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git # timeout=10
Fetching upstream changes from https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
using GIT_SSH to set credentials ghp credential
 > git fetch --tags --force --progress -- https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/feature/ansible^{commit} # timeout=10
 > git rev-parse feature/ansible^{commit} # timeout=10
Checking out Revision c7eb07cf70f930791fe26da949a502eaf8f42880 (refs/remotes/origin/feature/ansible)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f c7eb07cf70f930791fe26da949a502eaf8f42880 # timeout=10
Commit message: "Fix security warning for pem File"
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (copy files to ansible server)
[Pipeline] script
[Pipeline] {
[Pipeline] echo
copying all files to ansible control node
[Pipeline] sshagent
[ssh-agent] Using credentials root
[ssh-agent] Looking for ssh-agent implementation...
[ssh-agent]   Exec ssh-agent (binary ssh-agent on a remote machine)
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-avCjM73SRVoS/agent.1309
SSH_AGENT_PID=1311
Running ssh-add (command line suppressed)
Identity added: /var/jenkins_home/workspace/ansible-pipeline@tmp/private_key_15354759313785232833.key (/var/jenkins_home/workspace/ansible-pipeline@tmp/private_key_15354759313785232833.key)
[ssh-agent] Started.
[Pipeline] {
[Pipeline] sh
+ scp -o StrictHostKeyChecking=no ansible/ansible.cfg ansible/docker-compose.yaml ansible/hosts ansible/inventory_aws_ec2.yaml ansible/my-playbook.yaml root@139.59.167.35:/root
[Pipeline] withCredentials
Masking supported pattern matches of $keyfile
[Pipeline] {
[Pipeline] sh
+ scp **** root@139.59.167.35:/root/ssh-key.pem
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 1311 killed;
[ssh-agent] Stopped.
[Pipeline] // sshagent
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

## Executing Ansible Playbook

I have all the necessary files copied and moved to Ansible server, the next thing anticipated is execution of the playbook from Jenkins server by triggering the remote server. The complete Jenkinsfile script will be as follows.

```Jenkinsfile

pipeline {
    agent any
    stages {
        stage("copy files to ansible server") {
            steps {
                script {
                    echo "copying all files to ansible control node"
                    sshagent(['ansible-server-key']) {
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@139.59.167.35:/root"

                        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            sh 'scp $keyfile root@139.59.167.35:/root/ssh-key.pem'
                        }
                    }
                }
            }
        }
        stage("execute ansible playbook with EC2 Instances") {
            steps {
                script {
                    echo "calling ansible playbook to configure EC2 instances"
                    def remote = [:]
                    remote.name = "ansible-server"
                    remote.host = "139.59.167.35"
                    remote.allowAnyHosts = true

                    withCredentials([sshUserPrivateKey(credentialsId: 'ansible-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]){
                        remote.user = user
                        remote.identityFile = keyfile
                        sshCommand remote: remote, command: "ansible-playbook my-playbook.yaml"
                    }
                }
            }
        }
    }
}
```

After executing, the build threw some errors and the main reason is that I did have stopped EC2 instances in AWS and Ansible server was trying to write into them. In order to get the code to pass, instead of declaring **all** in the playbook host files, you may want to add the server IP addresses.

![Jenkins-7](./images/image-11.png)

![Jenkins-8](./images/image-11-b.png)

![Jenkins-9](./images/image-11-c.png)

I have Docker and Docker-compose installed in AWS two EC2 instances, and nginx is running in a docker container as anticipated.

![Jenkins-10](./images/image-12.png)

Jenkins console output

```shell
Started by user Joseph
Lightweight checkout support not available, falling back to full checkout.
Checking out git https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git into /var/jenkins_home/workspace/ansible-pipeline@script to read Jenkinsfile
The recommended git tool is: NONE
using credential Github-GCP
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/ansible-pipeline@script/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git # timeout=10
Fetching upstream changes from https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
using GIT_SSH to set credentials ghp credential
 > git fetch --tags --force --progress -- https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/feature/ansible^{commit} # timeout=10
 > git rev-parse feature/ansible^{commit} # timeout=10
Checking out Revision 23412fcab2b05ed7328c8b74b9dc28ff8e27ad3f (refs/remotes/origin/feature/ansible)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 23412fcab2b05ed7328c8b74b9dc28ff8e27ad3f # timeout=10
Commit message: "Refactored File"
 > git rev-list --no-walk 23412fcab2b05ed7328c8b74b9dc28ff8e27ad3f # timeout=10
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/ansible-pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
The recommended git tool is: NONE
using credential Github-GCP
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/ansible-pipeline/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git # timeout=10
Fetching upstream changes from https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
using GIT_SSH to set credentials ghp credential
 > git fetch --tags --force --progress -- https://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@github.com/appwebtech/Ansible-Integration-Jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/feature/ansible^{commit} # timeout=10
 > git rev-parse feature/ansible^{commit} # timeout=10
Checking out Revision 23412fcab2b05ed7328c8b74b9dc28ff8e27ad3f (refs/remotes/origin/feature/ansible)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 23412fcab2b05ed7328c8b74b9dc28ff8e27ad3f # timeout=10
Commit message: "Refactored File"
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (copy files to ansible server)
[Pipeline] script
[Pipeline] {
[Pipeline] echo
copying all files to ansible control node
[Pipeline] sshagent
[ssh-agent] Using credentials root
[ssh-agent] Looking for ssh-agent implementation...
[ssh-agent]   Exec ssh-agent (binary ssh-agent on a remote machine)
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-7HIeUhxx4whB/agent.2069
SSH_AGENT_PID=2071
Running ssh-add (command line suppressed)
Identity added: /var/jenkins_home/workspace/ansible-pipeline@tmp/private_key_8120171997839084604.key (/var/jenkins_home/workspace/ansible-pipeline@tmp/private_key_8120171997839084604.key)
[ssh-agent] Started.
[Pipeline] {
[Pipeline] sh
+ scp -o StrictHostKeyChecking=no ansible/ansible.cfg ansible/docker-compose.yaml ansible/hosts ansible/inventory_aws_ec2.yaml ansible/my-playbook.yaml root@139.59.167.35:/root
[Pipeline] withCredentials
Masking supported pattern matches of $keyfile
[Pipeline] {
[Pipeline] sh
+ scp **** root@139.59.167.35:/root/ssh-key.pem
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 2071 killed;
[ssh-agent] Stopped.
[Pipeline] // sshagent
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (execute ansible playbook with EC2 Instances)
[Pipeline] script
[Pipeline] {
[Pipeline] echo
calling ansible playbook to configure EC2 instances
[Pipeline] withCredentials
Masking supported pattern matches of $keyfile
[Pipeline] {
[Pipeline] sshCommand
Executing command on ansible-server[139.59.167.35]: ansible-playbook my-playbook.yaml sudo: false

PLAY [Install python3, docker, docker-compose] *********************************

TASK [Install python3 and docker] **********************************************
ok: [ec2-46-137-34-225.eu-west-1.compute.amazonaws.com]
ok: [ec2-18-203-99-123.eu-west-1.compute.amazonaws.com]

TASK [Install Docker-compose] **************************************************
ok: [ec2-46-137-34-225.eu-west-1.compute.amazonaws.com]
ok: [ec2-18-203-99-123.eu-west-1.compute.amazonaws.com]

TASK [Start docker daemon] *****************************************************
ok: [ec2-46-137-34-225.eu-west-1.compute.amazonaws.com]
ok: [ec2-18-203-99-123.eu-west-1.compute.amazonaws.com]

TASK [Install docker python module] ********************************************
ok: [ec2-18-203-99-123.eu-west-1.compute.amazonaws.com]
ok: [ec2-46-137-34-225.eu-west-1.compute.amazonaws.com]

PLAY RECAP *********************************************************************
ec2-18-203-99-123.eu-west-1.compute.amazonaws.com : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ec2-46-137-34-225.eu-west-1.compute.amazonaws.com : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```