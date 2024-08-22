# 02 - Operating Systems & Linux Basics

# Course Documents

[02 - Linux Checklist.pdf](https://drive.google.com/file/d/1QLdVRcs5lVE3kPEasbPPFC3rnMwa9ACs/view?usp=drivesdk)

[02 - Linux Handout.pdf](https://drive.google.com/file/d/1QIcbuVXONGk055rBOUvCemLEntJSFNgY/view?usp=drivesdk)

# File System

- /bin contains system-wide binaries that are available to all users
- /sbin contains system binaries that are system related and need superuser access
- /usr this was used for user home directories before.
    - most of the folders here are linked from /bin
- /usr/local programs that we (user) install on the computer
    - (third-party apps like docker, minikube, java,…)
    - programs that split its components (binaries, libs, configs,..)
    - all programs here are available to all users
- /opt also for third-party apps like /usr/local but for programs that do not split its components
    - all programs here are available to all users
- /boot contains files required for booting the system
- /etc system-wide configuration (network config, user passwords, )
- /dev location of device files (hard disks, keyboard,…)
- /var contains files to which the system writes data during the course of its operation
    - /var/log contains log files
    - /var/cache contains cached data from appls
- /tmp temporary resources required for some processes, kept here temporarily
- /media contains subdirectories that represent mounted, removable devices like a cdrom, usb
- /mnt temporary mount points. historically, sys admins mounted temp file systems here

# CLI

- $ represents a regular user
- # represents the root user
- $ history shows the history of commands executed
- $ Ctrl+r allows to search previous commands
- $ uname -a shows info about OS
- $ cat /etc/os-release shows info about the OS release
- $ lscpu shows info about the cpu
- $ lsmem shows info about the RAM
- $ su - admin change to user admin

## Pipes & Redirects

### Input and Output

- output from one program can become the input of another command
- "pipe" command: | pipes the output of the previous command as an input to the next command
- Example:
    - $ cat /var/log/syslog | less
        - pipes the output of cat to less as input
        - cat shows the entire file in the command line, less displays one page at a time
    - less allows to navigate forward and backward through the file
    - less does't need to read the entire input file so it is faster with larger files
    - space advances one page, b goes back one page. q to quit

### Pipe & Less

- less allows to navigate forward and backward through the file
- less does't need to read the entire input file so it is faster with larger files
- space advances one page, b goes back one page. q to quit
- examples:
    - ls /usr/bin | less
    - history | less

### Pipe & Grep

- grep searches for a pattern of characters and displays all lines that contain that pattern
- example:
    - history | grep sudo
        - filter and display all lines that contain the word “sudo” that history command outputs
    - history | grep "sudo chmod"
        - filter and display all lines that contain the string “sudo chmod” that history command outputs
        - since it's not a single word it needs the double quotes
    - history | grep sudo | less
        - run history command, pipe its output to grep to filter for the "sudo" word and pipe the output of grep to less
    - ls /usr/bin | grep java
        - filter the ls for the "java" word
        - in this case if it prints java as output means we have the java binary
    - cat Documents/java-app/config.yaml | grep ports
        - filter the content of config.yaml and see if it contains the "ports" word

### Redirects in Linux

- > character is the redirect operator
- takes the output from the previous command and sends it to a file that we give
- example
    - $ history | grep sudo > sudo-commands.txt
        - takes the output of history | grep sudo and puts it into a file named sudo-commands.txt
        - if the file already exists it will be overwritten
    - $ cat sudo-commands.txt > sudo-rm-commands.txt
        - creates a new file named sudo-rm-commands.txt with the same content
        - to append content to a file with redirect we need to use >>
        - example:
            - $ history | grep rm >> sudo-rm-commands.txt
                - appends to the end of sudo-rm-commands.txt the result of history | grep rm

### Wrap Up & Standard I/O

**Pipes**

Command | Command

**Redirects**

Command > Command

**Standard I/O**

- every program has 3 built-in streams:
    - STDIN (0) = Standard Input
    - STDOUT (1) = Standard Ouptut
    - STDERR (2) = Standard Error
- We pipe or redirect the standard output from one command to the standard input of another command
    - Command → STDOUT → STDIN → Command → STDOUT
    
- We can run multiple commands in one line without sharing output and input using ;
- Example:
    - $ clear; sleep 1; echo "Hi”
        - clear the screen, sleep for 1 second and print the message

# Package Manager

**APT**

- $ sudo apt update updates the package index
- $ sudo apt search <package_name> search for a given package
- $ sudo apt install <package_name> install a given package
- $ sudo apt install <package_name1> <package_name2> install multiple packages
- $ sudo apt remove <package_name> removes installed package
- /etc/apt/sources.list is a file that contains the repositories that apt uses
- $ sudo add-apt-repository "repository” adds a new repository to the list

**APT-GET**

- There is also apt-get package manager

**SNAP**

- Snap is another package manager that is used for installing single bundled applications like vs code.
- $ sudo snap install --classic code

**Repositories**

- PPA (Personal Package Archive) are repositories provided by the community. Anybody can create a PPA to distribute software
    - This is not official repositories

Use apt when possible, than snap and than add repository

Debian based distros use apt or apt-get, Red Hat based distros use yum

# VIM Editor

- Vim has 2 modes
    - Command Mode
        - default mode
        - can't edit text
        - what is typed is interpreted as a command
        - We can navigate, search, delete, undo, etc
    - Insert Mode
        - allows to enter text
- pressing 'i' switches to insert mode
- pressing 'Esc' switches to command mode
- to write the file to disk and quit the editor: ':wq’ (write and quit)
- to exit the editor and discard the changes: ':q!’
- to delete an entire line in command mode: 'dd’
- to delete the next 10 lines: 'd10d’
- to undo a change type : 'u' (we can keep issuing more undos)
- to go to the end of the current line and switch to insert mode: 'A’
- to go to the end of the current line and NOT switch to insert mode: '$’
- to go to the beginning of the line type '0' (this one will keep in command mode)
- jump to a specific line: '12G’
- to search for strings in command mode type '/pattern’ and press enter
    - Example: /nginx
    - type 'n' to jump to the next match
    - type 'N' to jump to the previous match
- to replace a string with a new value: ':%s/old/new’
    - Example to replace all occurrences of nginx with web-app: ':%s/nginx/web-app’

# Users & Permissions

## Linux Accounts & Groups

**3 User Categories**

- Superuser Account
    - root user - unrestricted permissions
    - there is only one root user per machine
- User Account
    - a regular user we create to login
    - each regular user has it's home directory inside /home/
- Service Account
    - relevant in server distros
    - each service will get its own user (e.g. mysql user will start mysql application)
    

**Permissions**

- User Level
    - give permissions to user directly
- Group Level
    - group users into Linux Groups
    - give permissions to the group
    - A user can be on multiple groups

- /etc/passwd stores user account info (everyone can read but only root can change the file)
    - each line represents a user
    - Format used in each line:
        - USERNAME:PASSWORD:UID:GID:GECOS:HOMEDIR:SHELL
    - Example line: nmartins:x:1000:1000:Nuno Martins,,,:/home/nmartins:/bin/bash
        - Username: nmartins
        - Password: x - means that encrypted password is in /etc/shadow file
        - User ID (UID): 1000 - each user has a unique ID. UID 0 is reserved for root
        - Group ID (GID): 1000 - the primary group ID (stored in /etc/group file)
        - User ID Info: Nuno Martins,,, - comment field for extra info about users (fullname, room number, phone, etc.)
        - Home directory: /home/nmartins - absolute path of user's home directory
        - Default Shell: /bin/bash - absolute path of a shell
    - DO NOT modify this file manually, use adduser instead
    
    To add a new user: sudo adduser tom
    
    this will create the user, group and home directory and add the entry to /etc/passwd file
    
    by default the group created has the same name as the user and it is it's default group
    
    To change a password of a user: sudo passwd <user_name>
    
    There's no need to know the previous password
    
    To add a new group: sudo groupadd <group_name>
    
    by default assigns the next GID from the range of IDS in the login.defs file
    
    The groups can be viewed in /etc/group file
    
    adduser and groupadd ???
    
    - adduser and addgroup
        - interactive
        - more user friendly
        - chooses conformant UID and GUID values
        - creates home dir with skeletal config automatically
        - or asks for input in interactive mode
        - Use these when executing manually
    - useradd and groupadd
        - we need to provide the infos
        - low-level utilities
        - use these in automation like scripts
        
    
    change the primary group of a user
    
    - sudo usermod -g devops tom (changes the primary group of user tom to devops)
    - usermod [OPTIONS] <username> : modify user account
    
    delete a group: sudo delgroup tom
    
    add a user to supplementary groups
    
    - sudo usermod -G <group1,group2> <username>
    - example: sudo usermod -G admin tom
    - the -G option overrides the user groups
        - To instead append a group to the already existing list of groups use -a:
            - sudo usermod -aG newgroup tom
            
    
    display the groups of a user
    
    - $ groups
        - show the groups that the current logged in user belongs to
    - $ groups tom
        - shows the groups that tom belongs to
    
    Remove a user from a group
    
    - sudo gpasswd -d nicole devops
    

## File Ownership & Permissions

- user permissions are related to reading, writing and executing files

### **File Ownership**

- Who owns the file/directory?

- Ownership levels of a file/directory
    - Which user owns the file?
    - Which group owns the file?
- ls -l shows the user that owns the file and the owning group
- the owner is the user who created the file
- the owning group is the primary group of that user
- to change the ownership of a file (both user and group):
    - chown <username>:<groupname> <filename>
    - example: sudo chown tom:admin test.txt
- to change only the user that owns the file:
    - chown <username> <filename>
    - example: sudo chown admin test.txt
- to change only the group owner:
    - chgrp <groupname> <filename>
    - example: sudo chgrp devops test.txt

### **File Permissions**

- ls -l shows the permissions of a file/directory
- permissions examples:
    - drwxrwxr-x (directory example)
    - -rw-rw-r-- (file example)
- The first char in blue represents the file type
    - - regular file
    - d directory
    - c character device file
    - l symbolic link
    - etc.
- The next 3 chars in green represent the user owner permissions
    - r Read
    - w Write
    - x Execute
    - - No permission (the permission is absent)
- The next 3 chars in pink represent the group owner permissions
    - r Read
    - w Write
    - x Execute
    - - No permission (the permission is absent)
- The next 3 chars in purple represent all other users permissions (who are not the file owner or don't belong to the group owner)
    - r Read
    - w Write
    - x Execute
    - - No permission (the permission is absent)

- remove execute permission from api folder (for all owners)
    - sudo chmod -x api (- indicates to remove a permission)
        - the same could be done: sudo chmod a-x api (a stands for all, - indicates to remove a permission)
    - permissions before the command executed: drwxrwxr-x
    - permissions after command executed: drw-rw-r--
- remove the write permission from the group on config.yaml file
    - sudo chmod g-w config.yaml (g stands for group)
    - permissions before the command executed: -rw-rw-r--
    - permissions after the command executed: -rw-r--r--
- add the execute permission to the group on config.yaml file
    - sudo chmod g+x config.yaml (+ indicates to add a permission)
    - permissions before the command executed: -rw-r--r--
    - permissions after the command executed: -rw-r-xr--
- add the execute permission to the user on script.sh file
    - sudo chmod u+x [script.sh](http://script.sh) (u stands for user)
    - permissions before the command executed: -rw-rw-r--
    - permissions after the command executed: -rwxrw-r--
- add the execute permission to others on script.sh file
    - sudo chmod o+x [script.sh](http://script.sh) (o stands for others)
    - permissions before the command executed: -rwxrw-r--
    - permissions after the command executed: -rwxrw-r-x
- we can also specify in one go all permissions for the group
    - sudo chmod g=rwx config.yaml
    - permissions before the command executed: -rw-r--r--
    - permissions after the command executed: -rw-rwxr--
    - it also works for user (u), others (o) and all (a)
- Changing permissions through Absolute(Numeric) Mode
    
    
    | Number | Permission Type | Symbol |
    | --- | --- | --- |
    | 0 | No Permission | --- |
    | 1 | Execute | --x |
    | 2 | Write | -w- |
    | 3 | Execute + Write | -wx |
    | 4 | Read | r-- |
    | 5 | Read + Execute | r-x |
    | 6 | Read + Write | rw- |
    | 7 | Read + Write + Execute | rwx |
    - give everyone read, write and execute permissions
        - sudo chmod 777 script.sh
        - permissions before the command executed: -rw-rw-r--
        - permissions after the command executed: -rwxrwxrwx
    - give user all permissions, group read permission and others no permissions:
        - sudo chmod 740 script.sh
        - permissions before the command executed: -rwxrwxrwx
        - permissions after the command executed: -rwxr-----

# Shell Scripting

## Introduction

- avoid repetitive work
- keep history of configuration
- share the instructions
- logic & bulk operations
- write commands in a file
- execute that file
- such file is called a shell script
- shell scripts have .sh file extension
- Different shells
    - sh (Bourne Shell) - /bin/sh
        - used to be the default
    - Bash (Bourne again shell) - /bin/bash
        - improved version of sh
        - default shell for most UNIX like systems

## Shebang

- All shell script files have the same .sh file extension
- we need to tell the OS which shell to use with a shebang:
    - #!/bin/sh
    - #!/bin/bash
    - #!/bin/zsh
- Why is it called "shebang"?
    - because of the first 2 characters: "#!”
    - # = in musical notation, also called "sharp”
    - ! = also called "bang”
    - shebang become a shortening of sharp-bang
    

## Write & Execute Simple Script

$ vim setup.sh

```bash
#!/bin/bash

echo "Setup and configure server"
```

give execute permission to the user: $ sudo chmod u+x setup.sh

to execute the file: $ ./setup.sh

or $ bash setup.sh

## Variables

```bash
#!/bin/bash

echo "Setup and configure server"

file_name=config.yaml
config_files=$(ls config)

echo "using file $file_name to configure something"
echo "here are all configuration files: $config_files"
```

- file_name=config.yaml is a variable
- config_files=$(ls config) is also a variable that stores the result of a command (ls config)

## if-else Statements

```bash
#!/bin/bash

echo "Setup and configure server"

file_name=config.yaml

if [ -d "config" ]
then
  echo "reading config directory contents"
  config_files=$(ls config)
else
  echo "config dir not found. Creating one"
  mkdir config
fi

echo "using file $file_name to configure something"
echo "here are all configuration files: $config_files"
```

- Square brackets enclose expressions
- It's a shorthand notation for the "test" command
    - if test -d "config" is the same
- -d is to check if it is a directory

## Basic Operators

## Passing Arguments to a Script

- Arguments passed to a script are processed in the same order in which they're sent
- the index of arguments starts at 1
- Parameters start from $1 to $9
- Reuse the same script with different parameters

```bash
#!/bin/bash

echo "Setup and configure server"

file_name=config.yaml

config_dir=$1

if [ -d "$config_dir" ]
then
  echo "reading config directory contents"
  config_files=$(ls "$config_dir")
else
  echo "config dir not found. Creating one"
  mkdir "$config_dir"
  touch "$config_dir/config.sh"
fi

user_group=$2

if [ "$user_group" == "nmartins" ]
then
  echo "configure the server"
elif [ "$user_group" == "admin" ]
then
  echo "administer the server"
else
  echo "No permission to configure server. wrong user group"
fi

echo "using file $file_name to configure something"
echo "here are all configuration files: $config_files"
```

## Read User Input

```bash
#!/bin/bash

echo "Reading user input"

read -p "Please enter your password: " user_pwd

echo "thanks for your password $user_pwd"
```

- we can ask user input with read and using -p to put text into the prompt

```bash
#!/bin/bash

echo "all params $*"
echo "number of params: $#"

echo "user $1"
echo "group $2"
```

- $* = represent all the arguments as a single string
- $# = total number of arguments provided

## Shell Loops

- types of loops
    - while loop
    - for loop
    - until loop
    - select loop
    

### for loop

```bash
#!/bin/bash

echo "all params $*"
echo "number of params: $#"

for param in $*
do
  if [ -d "$param" ]
  then
    echo "executing scripts in the config folder"
    ls -l "$param"
  fi

  echo $param
done
```

- operates on lists of items
- repeats a set of commands for every item in the list

### while loop

```bash
#!/bin/bash

sum=0
while true
do
  read -p "enter a score: " score

  if [ "$score" == "q" ]
  then
    break
  fi

  sum=$(($sum+$score))
  echo "total score: $sum"
done
```

- execute a set of commands repeatedly until some condition is matched
- double parenthesis for arithmetic operations
    - $(( 2 + 4 ))          $(( $num1 + $num2 ))

## Functions

```bash
#!/bin/bash

echo "all params $*"
echo "number of params: $#"

for param in $*
do
  if [ -d "$param" ]
  then
    echo "executing scripts in the config folder"
    ls -l "$param"
  fi

  echo $param
done

function score_sum {
  sum=0
  while true
  do
    read -p "enter a score: " score

    if [ "$score" == "q" ]
    then
      break
    fi

    sum=$(($sum+$score))
    echo "total score: $sum"
  done
}

score_sum
```

### Pass Parameters to a Function

```bash
#!/bin/bash

# echo "all params $*"
# echo "number of params: $#"

for param in $*
do
  if [ -d "$param" ]
  then
    echo "executing scripts in the config folder"
    ls -l "$param"
  fi

  echo $param
done

function score_sum {
  sum=0
  while true
  do
    read -p "enter a score: " score

    if [ "$score" == "q" ]
    then
      break
    fi

    sum=$(($sum+$score))
    echo "total score: $sum"
  done
}

function create_file() {
  file_name=$1
  is_shell_script=$2
  touch $file_name
  echo "file $file_name created"

  if [ "$is_shell_script" = true ]
  then
    chmod u+x $file_name
    echo "added execute permission"
  fi
}

create_file test.txt

create_file myfile.yaml

create_file myscript.sh true
```

- The parameters passed are represented by $1, $2, etc.
- Comments are represented by #

### Return Values from Functions

```bash
#!/bin/bash

function sum() {
  total=$(($1+$2))
  return $total
}

sum 2 10
result=$?

echo "sum of 2 and 10 is $result"
```

- $? captures value returned by last command

# Environment Variables

- Each user has its own environment
- Each user can configure its own environment/account by setting preferences
- These OS configs should be isolated from other user environments
- key-value pairs
- Environment variables are available for the whole environment
- Convention is to use UPPER CASE on the keys
- Examples
    - SHELL → default shell program location of the user
    - HOME → current user's home dir
    - USER → currently logged in user
    

## List all Environment Variables

$ printenv → displays all env variables that the OS sets for the current user

## Print Specific Environment Variables

$ printenv USER

$ printenv | grep USER

$ echo $USER

## Application Environment Variables

- OS stores info about the environment
- we can create our own environment variables
- Use case: Sensitive data for application
    - We have an app that connects to a database
    - We need to store the login credentials of the database
    - We can use env vars and the app can read those env vars
        - DB_PWD=secretpassword
        - DB_USER=mysqluser
    - These env vars are available in the environment
    - all apps and processes can now access these env vars
    - Every programming language allows to access env vars

## Creating Environment Variables

$ export DB_USERNAME=dbuser

$ export DB_PASSWORD=secretpwdvalue

$ export DB_NAME=mydb

$ printenv | grep DB → will print the above env vars

We can access the env vars with echo $DB_NAME for example

We can change the values by exporting again like: export DB_NAME=newdbname

## Deleting Environment Variables

$ unset DB_NAME

## Persisting Environment Variables

- When we set env vars in the command line they are only available in that session. If we close the command line they are gone!
- ~/.bashrc file in the user's home directory
    - shell specific configuration file
    - per-user shell specific config file
    - variables set in this file are loaded whenever a bash login shell is entered
    - When we change the .bashrc file the system does not no the change, so we need to source the file: $ source ~/.bashrc
    - After this the changes are permanent and the shell can be closed and the env vars will still be there

## Persisting Environment Variables (system wide)

- /etc/environment file
    - contains the PATH env var

### PATH Environment Variable

- list of directories to executable files, separated by :
- tells the shell which directories to search for the executable in response to our executed command
- Instead of running $ /usr/bin ls -l we can just run $ ls -l
    - Because /usr/bin is in the PATH env var
    

### Add a custom command/program

- Create following file in home dir and call it welcome

```bash
#!/bin/bash

echo "Welcome to DevOps Bootcamp $USER"
```

- change its permissions:
    - $ sudo chmod a+x welcome
- Edit ~/.bashrc file and add the following:
    - PATH=$PATH:/home/nmartins
- source the file
    - $ source ~/.bashrc
- From any location just type ‘welcome' and it will run

# Networking

## LAN, Switch, Router, WAN, Gateway

### LAN

- Local Area Network
- collection of devices connected together in one physical location
- each device has a unique IP address
- Examples: home, school, company
- Devices communicate via these IPs

### Switch

- sits within the LAN
- facilitates the connection between all the devices within the LAN

### Router

- sits between LAN and outside networks (WAN)
- connects devices on LAN and WAN
- allows networked devices to access the internet

### Gateway

- IP address of the router

## Subnet

- devices in the LAN belong to same IP address range
- Subnet = logical subdivision of an IP network
- Subnetting = process of dividing a network into two or more networks
- Example of IP address range:
    - IP address: 192.168.0.0 → starting point of IP range, the first IP in the range
    - Subnet Mask: 255.255.255.0 → sets the IP range
- 255.255.0.0 → means that 16 bits are fixed
- 255.255.255.0 → means 24 bits are fixed
- CIDR → Classless Inter-Domain Routing → Shorthand
    - 192.168.0.0/16 → /16 bits are fixed
    - 192.168.0.0/24 → /24 bits are fixed
    

## Network Address Translation (NAT)

- IP adress within LAN are not visible to the outside network or internet
- when you send a request to a server on the internet your pc IP address is replaced by the router
- Benefits of NAT
    - Security and protection of devices within LAN
    - Reuse IP addresses
    - Limited number of IPv4 addresses available:
        - 4,294,967,296
        

## Firewall

- by default the server is not accessible from outside the LAN
- is a system that prevent unauthorised access from entering a private network
- using firewall rules you can define, which requests are allowed
    - which IP address in your network is accessible
    - which IP address can access your server
    - we can allow any device also

## Port

- every device has a set of ports
- you can allow specific ports
- you can allow specific ports AND specific IP address
- different apps listen on specific ports
- Standard ports for many apps
    - Port 80 → web servers
    - Port 3306 → MySQL DB
    - Port 5432 → PostgreSQL DB
- for every app you need a port
- each port is unique on a device

## Domain Name System (DNS)

- Mapping IP addresses to Names
- when you type a url like [www.facebook.com](http://www.facebook.com) it gets translated to an IP address
    - the service that does this translation is DNS
        - translates domain names to IP Addresses
- Domain names have a hierarchical structure
    - Root Domains
        - Top Level Domains (.mil, .edu, .com, .org, .net, .gov)
        - Top Level Domains (.at, .de, .us, .fr)
            - you can buy a website like mywebsite.com
- This is all managed by ICANN (Internet Corporation for Assigned Names and Numbers)
    - Manages the TLD development and architecture of the internet domain space
    - Authorises Domain Name Registrars, which register and assign domain names

| Root | . |
| --- | --- |
| Top Level Domain Name | .com |
|  | techworld-with-nana |
| Subdomain
(this could be 3 different servers/apps) | bootcamp → www.bootcamp.techworld-with-nana.com
workshops  → www.workshops.techworld-with-nana.com
courses  → www.courses.techworld-with-nana.com |
- every computer has a DNS client pre installed
- You type [www.facebook.com](http://www.facebook.com) → DNS Client → Resolver (usually your Internet Service Provider). if it does not have in cache ir goes to one of the Root servers
- Root Server
    - requests for top level domains
    - placed around the world
- both PC and Resolver will cache entries

## Networking Commands

- ifconfig → info about IP addresses, gateway, mask, etc
- netstat → shows active connection on the machine
- ps aux → check current running processes (includes the port and computer resources)
- nslookup → can get an IP address of any domain name
- ping → checks if a service or app is accessible

# SSH - Secure Shell

- SSH is a network protocol that gives users a secure way to access a computer over the internet
- SSH refers also to the suite of utilities that implement that protocol
- 2 ways to authenticate over SSH
    - Username & Password
        - the user account is previously configured on the server
    - SSH Key Pair
        - Client creates an SSH Key Pair
            - Key Pair = Private Key + Public Key
        - Private Key = Secret key. Is stored securely on the client machine
        - Public Key = Public. Can be shared, e.g. with the remote server
        - By putting the public key into the remote server the server is able to trust connections from the client
        - Client can "unlock" the public key with his private key
        

## SSH for Services

- Services, like Jenkins often need to connect to another server via 
SSH
    - Create Jenkins User on application server
    - Create SSH Key Pair on Jenkins server
    - Add public SSH key to authorized_keys on application server
    

## Firewall and Port 22

- Communication must be explicitly be allowed through Firewall rule
- By default it's blocked
- SSH Authentication comes AFTER the connection
- SSH Service runs by default on machine
- By default, SSH server listens on port 22
- In Firewall we allow access on port 22
- SSH is powerful and needs to be restricted to specific IP addresses

## SSH in Action

1. Create Remote Server on Cloud Platform
    1. Created the most basic instance and a password for root user
    2. connected via ssh with: $ ssh root@xxx.xxx.xxx.xxx
    3. need to provide root password and we are in!
2. Generate SSH Key Pair on our laptop
    1. $ ssh-keygen -t rsa
        1. rsa is the type of key pair we are creating
    2. $ ls .ssh/
        1. id_rsa → Private Key
        2. id_rsa.pub → Public Key
3. Add Public Key to authorized_keys
    1. On the remote server the should be a file /root/.ssh/authorized_keys
    2. This is the file where we add all the public keys of all clients we want to access the server
    3. Copy the content of id_rsa.pub to the authorized_keys file
    4. exit the server and connect again using: $ ssh root@xxx.xxx.xxx.xxx
    5. $ ssh root@xxx.xxx.xxx.xxx is equivalent to $ ssh -i ssh/id_rsa root@xxx.xxx.xxx.xxx
        1. so if we have a key that is NOT named id_rsa we must specify its path
4. Copy Bash Script File to the Remote Server
    1. Create a [test.sh](http://test.sh) on the local machine with some code
    2. scp [test.sh](http://test.sh) root@xxx.xxx.xxx.xxx:/root
        1. allows to securely copy files and directories (secure meaning files and password are encrypted
        2. copies local [test.sh](http://test.sh) to remote server into /root dir
        3. this also takes the private key, but as it is the default id_rsa no need to specify
5. Execute Script on Remote Server
    1. going back to the remote server: $ ssh root@xxx.xxx.xxx.xxx 
    2. the [test.sh](http://test.sh) file will be there
    3. give execute permissions: 
        1. $ chmod u+x test.sh
    4. Run the script!:
        1. $ ./test.sh

# Action Items

- [x]  Search what a Hypervisor is (is Virtualbox an Hypervisor?) type 1 and type 2
- [ ]  Get a book or tutorial on Vim editor and learn more on how to use it. Also find a Cheat Sheet
    - [ ]  Neovim ??