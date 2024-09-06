# 05 - Cloud & IaaS Basics

# Intro to Cloud & IaaS

## What is Infrastructure as a Service?

- "The other way”
    - Company buys it's own servers (physical machines)
    - They manage the servers and infrastructure
    - if something breaks, we need to fix it
- The IaaS way
    - Delegate Infrastructure Management
    - Move your physical infrastructure to cloud
    - We just rent the servers

## Infrastructure as a Service Providers

They manage the infrastructure for us and we rent servers from them

- AWS
- Google Cloud
- Microsoft Azure
- DigitalOcean
- …

# Setup Server on DigitalOcean

For the demo I've chosen:

- Ubuntu 22.04 x64
- Basic Shared CPU
    - Regular with SSD
        - 512MB / 1CPU
        - 10 GB SSD
        - 500GB transfer
        - $4/m → $0.006/h
- Frankfurt location
- Authentication
    - The root user is created by default and we can setup the password
    - We are going with SSH key

After creating the Droplet, by default, everything is closed for external connections

To enable SSH access for the selected droplet:

- Networking → Firewalls → Edit → Create Firewall
    - Firewalls are used accross all DO, so can be used for multiple Droplets
    - Name: my-droplet-firewall
    - Inbound Rules
        - SSH should be pre-defined already on port 22
        - For the Sources putted my real IP address (it might need to be updated when my IP changes)
- Click Create Firewall
- Select the created Firewall → Droplets tab → Add Droplets → Select the one created before → Add Droplet

Inbound → Incoming requests

Outbound → outgoing requests from server

To connect to the droplet: $ ssh root@IP-Address

## Install Java

- We will install Java 8 because it is required later for Nexus
- $ apt update
- $ apt install openjdk-8-jre-headless

# Deploy and run application artifact on Droplet

We will build a jar file locally, copy it to remote server and run the app on the remote server

- In local machine:
    - $ git clone [git@github.com](mailto:git@github.com):nanuchi/java-react-example.git
    - $ cd java-react-example
    - $ ./gradlew build
    - $ scp build/libs/java-react-example.jar root@Droplet-IP-Address:/root
- On the Remote Droplet
    - $ ls (on the root folder) and the jar file should be there
    - $ java -jar java-react-example.jar OR java -jar java-react-example.jar & to run in detached mode and use the console
        - $ ps aux | grep java → shows the running java process
        - We can install netstat
            - $ apt install net-tools
            - $ netstat -lpnt
                - shows active connections
                - one of them will be our java app and the port is shown there (7071)
        - The application will start and also log the port where the app is listening (7071)
        - We have a public IP address of the droplet but the app port(7071) is not open!
    - Got to DigitalOcean UI → Networking → Firewalls → Select our Firewall
    - Add a new Inbound rule:
        - Type: Custom
        - Protocol: TCP
        - Port Range: 7071
        - Sources: leave the default All IPv4 and All IPv6
        - Hit Save
- On a local browser open http://Droplet-IP-Address:7071
    - It will load the app UI

# Create and configure a Linux user on a cloud server

We shouldn't start services and apps with root user

- Create separate user for every app
- give it only the permissions it needs to run that app
- **Don't work with the root user!**

On the remote Droplet:

- $ adduser nmartins
    - set a password
- we want this user to be able to run commands as a root user, so we need to add it to the sudo group
    - $ usermod -aG sudo nmartins
    - $ su - nmartins → will change the user
    - $ exit → will logout nmartins user and return to root user
    - $ exit → running exit again will disconnect from the remote server
    - If we try to ssh with the new user it will fail:
        - $ ssh nmartins@Droplet-IP-Address
        - Permission denied (publickey)
        - We need to configure the user nmartins with the ssh key
    - $ ssh root@Droplet-IP-Address
    - $ su - nmartins
    - create .ssh folder in nmartins home dir (/home/nmartins)
        - $ mkdir .ssh
    - $ sudo vim .ssh/authorized_keys
        - paste the public key inside the file and save it
    - $ exit → logout user nmartins
    - $ exit → logout user root and disconnect from remote server
    - $ ssh nmartins@Droplet-IP-Address
        - This time works!!

# Exercises

This is my solution of the exercise 5 - Cloud & IaaS Basics - DigitalOcean of the [TechWorld with Nana DevOps Bootcamp](https://www.techworld-with-nana.com/devops-bootcamp)

## Exercise 0: Clone Git Repository

```bash
git clone git@gitlab.com:devops-bootcamp3/node-project.git
mv node-project twn-devops-bootcamp-project5
cd twn-devops-bootcamp-project5

# Created an empty repository on GitHub
# https://github.com/MuhamedElhamy/twn-devops-bootcamp-project5.git

git remote set-url origin https://github.com/MuhamedElhamy/https://github.com/MuhamedElhamy/twn-devops-bootcamp-project5.git.git
git branch -M main
git push -u origin main
```

## Exercise 1: Package NodeJS App

```bash
cd app
npm pack # Creates a packaged app (bootcamp-node-project-1.0.0.tgz)
```

## Exercise 2: Create a new server

Steps on Digital Ocean UI to accomplish this on 17-09-2022:

- Manage → Droplets → Create Droplet
    - I've selected the following resources:
        - Distribution: Ubuntu 22.04 x64
        - Plan: Shared CPU Basic
        - CPU Options: Regular with SSD
            - RAM: 512MB
            - CPU: 1
            - SSD: 10GB
            - Transfer: 500GB
        - Location: Frankfurt
        - Authentication:
            - SSH keys → New SSH Key → Paste content of my Public SSH Key → Give it a name → Add SSH Key
        - Hostname: twn-devops-bootcamp-ex05
- Hit Create Droplet
- Select the created Droplet → Firewalls → Edit → Create Firewall
    - Name: twn-devops-bootcamp-ex05
    - Inbound Rules:
        - Type: SSH
        - Protocol: TCP
        - Port Range: 22
        - Sources: I've added my real IP address
- In the "Apply to Droplets" select the created Droplet → Hit Create Firewall

## Exercise 3: Prepare server to run Node App

In the UI we can view and copy the Droplet IP Address, let's assume for this exercise it is 157.230.108.207

```bash
ssh root@64.225.110.207
apt update
apt install nodejs
nodejs -v # to verify installation
apt install npm
npm -v # to verify installation

# To not use the root account I created a dedicated user to run the app
adduser nodeapp
usermod -aG sudo nodeapp # Add the user to the sudo group
su - nodeapp # change to nodeapp user
mkdir .ssh

# To allow the new user(nodeapp) to login with the SSH key we need to add it's 
# Public SSH Key to the following file:
sudo vim .ssh/authorized_keys # Paste Public SSH Key in this file. Save and quit

exit # logout nodeapp user
exit # logout root user and disconnect from remote server

# Now you can login with nodeapp user using the SSH key:
ssh nodeapp@64.225.110.207
```

## Exercise 4: Copy App and package.json

From your local command line run the following:

```bash
# Make sure you are inside the 'app' folder
# The following command will copy the bootcamp-node-project-1.0.0.tgz 
# to the home directory of the nodeapp user
scp bootcamp-node-project-1.0.0.tgz nodeapp@64.225.110.207:~
```

From you nodeapp command line on the droplet run the following:

```bash
# Make sure you are in nodeapp home dir (/home/nodeapp)
ls # This should show the bootcamp-node-project-1.0.0.tgz copied before
tar xvzf bootcamp-node-project-1.0.0.tgz # extract the file content
mv package app # rename the extracted 'package' folder to 'app'
cd app
```

## Exercise 5: Run Node App

Still in the remote command line

```bash
# You sould be inside '/home/nodeapp/app'
npm install # This will install the needed dependencies

# Start the app in detached mode
node server.js &
# You will see a message like 'app listening on port 3000!'
# Take note of the port
```

## Exercise 6: Access from browser - configure firewall

In the DigitalOcean UI do the following:

- Networking → Firewalls → Select the one created before
    - Add a new Inbound Rules:
        - Type: Custom
        - Protocol: TCP
        - Ports: 3000 (the port you noted before when starting the app)
        - Sources: All IPv4 All IPv6
- Hit Save

Open you browser on http://64.225.110.207:3000
