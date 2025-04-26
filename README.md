# Ansible, Jenkins, and Docker Automation Project

This project demonstrates end-to-end automation using Ansible, Jenkins, Docker, and GitHub across three Ubuntu servers (Master, Test, and Production).
It automates software installation, server provisioning, Jenkins node setup, CI/CD pipelines, and web application deployment using Docker containers.

# Project Setup

Instances Created: 3 Ubuntu EC2 Instances (Master, Test, Production)
Keypair: ubuntu_ajith
Network: Default VPC and Security Groups

# Tools & Technologies Used

AWS EC2
Ansible
Jenkins
Docker
GitHub
Ubuntu Server 22.04

# Step-by-Step Process

1.Server Setup and Ansible Installation

  *Created 3 EC2 Instances: Master, Test, Production.
  *On Master instance, created a shell script ansible_install.sh with the following commands:
  
      sudo apt update
      sudo apt install software-properties-common
      sudo add-apt-repository --yes --update ppa:ansible/ansible
      sudo apt install ansible

# Ran the script:

      bash ansible_install.sh

# Edited the Ansible hosts file:

     sudo nano /etc/ansible/hosts
     Added Private IP addresses of Test and Production machines.
   
# Set up SSH keys on Master:

    ssh-keygen
    
* Copied public key to Test and Production servers:
  
   pasted the key manually into ~/.ssh/authorized_keys on the servers


2. Jenkins Installation on Master
 
*Created jenkins_install.sh with the following commands:

  
        sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2023.key
        echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
        sudo apt-get update
        sudo apt-get install jenkins -y
        sudo apt update
        sudo apt install openjdk-17-jdk -y
        sudo apt install docker.io -y

*For Test and Prod servers, created a dependencies.sh script:

        sudo apt update
        sudo apt install openjdk-17-jdk -y
        sudo apt install docker.io -y
  
3. Ansible Playbook for Automation
 
*Created install.yaml Ansible Playbook:

    ---
    - hosts: localhost
      become: true
      tasks:
      - name: Install Jenkins, Java, and Docker
        script: jenkins_install.sh

    - hosts: test
      become: true
      tasks:
      - name: Install Java and Docker on Test
        script: dependencies.sh

    - hosts: prod
      become: true
      tasks:
      - name: Install Java and Docker on Production
        script: dependencies.sh
      
*Checked server connectivity

    ansible -m ping all
   
*Ran the playbook:

    ansible-playbook install.yaml

4.Jenkins Master-Agent Setup

*Accessed Jenkins from browser:

    http://<master-ip>:8080

*Set up Jenkins nodes:

  Test Node and Production Node
  Remote Directory: /home/ubuntu/jenkins
  Host Key Verification: Non-Verifying
  Availability: Keep Agent online as much as possible

# Jenkins Jobs and CI/CD Pipelines

 Job 1: mastertesting_82 - Build on Master Branch Push
 
 Forked the GitHub repository: hshar/website ➔ ajithpethanan/website

On Master:

    git clone https://github.com/ajithpethanan/website.git

*Created a Dockerfile:

dockerfile:

    FROM ubuntu
    RUN apt update
    RUN apt install apache2 -y
    ADD . /var/www/html
    ENTRYPOINT apachectl -D FOREGROUND

*Added and committed Dockerfile:

     git add Dockerfile
     git commit -m "adding Dockerfile"

*Setup GitHub Webhook:

Payload URL:

     http://<master-ip>:8080/github-webhook/

*In Jenkins Build Steps (Execute Shell):

     sudo docker build /home/ubuntu/jenkins/workspace/mastertesting_82 -t masterimage
     sudo docker run -it -d --name c1 -p 82:80 masterimage

*After building, accessed via:

    http://<test-machine-ip>:82

# Job 2: testingdevelop_83 - Build on Develop Branch Push

*Created develop branch:

    git branch develop
    git checkout develop
    git push origin develop
    Made changes in index.html (example: "Hello Develop Testing").

*Git Commands:

    git add index.html
    git commit -m "adding new index.html"
    git push origin develop

# Jenkins Build Steps:

    sudo docker build /home/ubuntu/jenkins/workspace/testingdevelop_83 -t dockerimage
    sudo docker run -it -d -p 83:80 dockerimage

*Accessed via:

    http://<test-machine-ip>:83

# Job 3: finalrelease - Deploy to Production

This job triggers automatically after successful mastertesting_82.

# Jenkins Build Steps:

    sudo docker build /home/ubuntu/jenkins/workspace/finalrelease -t productimage
    sudo docker run -it -d -p 80:80 productimage

*Setup in mastertesting_82 configuration:

   Post Build Actions ➔ Build Finalrelease Job only if build is stable.

*Access via:

    http://<production-machine-ip>:80

Important Docker Commands Used:

# List Docker images

    sudo docker images

# List running containers

    sudo docker ps

# Build Docker image

    sudo docker build /path/to/dockerfile -t <image-name>

# Run Docker container

    sudo docker run -it -d --name <container-name> -p <host-port>:80 <image-name>




