# 06 - Artifact Repo Manager

# Intro

- Artifacts → apps built into a single file (JAR, WAR, ZIP, TAR, etc)
- Artifact repository → storage of those artifacts
- Different programming panguages produce different types of artifacts
- You need different repositories for them

## Artifact Repository Manager

- An Artifact Repository Manager manages different artifact repositories under 1 software
- Nexus is an Artifactory Repository Manager, like JFrog Artifactory
- upload and store different built artifacts
- retrieve (download) artifacts later
- central storage
- There is also public repository managers
    - libs and frameworks we use as dependencies
        - Maven Central Repository for Java
        - npm for JavaScript
- Nexus:
    - host own repositories
    - proxy repository
    - open source and commercial
    - multiple repositories for different formats
    - integrate with LDAP
    - REST API for integration
    - backup and restore
    - metadata tagging (labelling and tagging artifacts)
    - cleanup policies
    - search functionality
    - user token support for system user authentication

# Install and Run Nexus on a Cloud Server

- Droplet with 8GB of RAM with Java 8 installed

```bash
cd /opt
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz
ls
```

- There are 2 resulting folders from tar: nexus-x.y.x and sonatype-work
    - Nexus folder
        - contains runtime and Nexus application
    - sonatype-work
        - contains own config for Nexus and data. If we upgrade Nexus to a new version we keep the custom config and data
        - subdirectories depending on Nexus configuration
        - IP address that accessed Nexus
        - Logs of Nexus App
        - Your uploaded files and metadata
        - you can use this folder for backup
- Services should not run with root user permissions
- Best practice: create own user for service (e.g. Nexus)
- Only the permission for that specific service

```bash
adduser nexus
ls -l

# Nexus user needs permissions to execute nexus executable 
# and to read/write sonatype-work folder

# change ownership recursively for nexus user and nexus group
chown -R nexus:nexus nexus-3.72.0-04
chown -R nexus:nexus sonatype-work
ls -l
```

- Set Nexus configuration to run as nexus user

```bash
vim nexus-3.72.0-04/bin/nexus.rc

# uncomment the line and add nexus user, it should look like:
# run_as_user="nexus"
# Save and quit
```

- Run Nexus service as nexus user

```bash
su - nexus
/opt/nexus-3.72.0-04/bin/nexus start
ps aux | grep nexus # check that it is running
netstat -lnpt # match the PID with previous command to find the Nexus ports
```

- Add an Inbound Firewall rule to open port 8081 in DO UI
- open browser on [http://Droplet-IP-Address:8081](http://Droplet-IP-Address:8081)
- By default there is a admin user and the password can be viewed by:
    - cat /opt/sonatype-work/nexus3/admin.password
    - once you login on the wizard you must change the password

# Repository Types

## Proxy

- linked to a remote repository
- if artifact is in Nexus is retrieved from there, if not it will go to remote and add it to Nexus and retrieve
- Saves network bandwidth and time
- 1 single endpoint for repositories

## Hosted

- primary storage for company owned artifacts

## Group

- 

# Publish Artifact to Repository

## Upload Jar file to Nexus

- upload jar file to existing Nexus hosted repository on Nexus
- Maven/Gradle command for pushing to remote repository
- Configura both tools to connect to Nexus(Nexus Repo URL + Credentials)
- Nexus User with permission to upload

## Create Nexus User

In Nexus UI logged in as admin

- Security → Users → Create local user
    - Fill the fields, set status as Active and add role nx-anonymous
    - Hit Create User
- Roles → Create Role
    - Role Type: Nexus Role
    - Role ID: nx-java
    - Role Name: nx-java
    - Add privileges:
        - nx-repository-view-maven2-maven-snapshots-*
    - Hit Save
- Users → elhamy
    - add nx-java role
    - remove nx-anonymous role
    - Hit Save

## Gradle Project - Configure with Nexus

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.2.2.RELEASE'
    id 'io.spring.dependency-management' version '1.0.8.RELEASE'
}

group 'com.example'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

apply plugin: 'maven-publish'

publishing {
    publications {
        maven(MavenPublication) {
            artifact("build/libs/my-app-$version"+".jar") {
                extension 'jar'
            }
        }
    }

    repositories {
        maven {
            name 'nexus'
            url "http://64.225.108.229:8081/repository/maven-snapshots/"
            allowInsecureProtocol = true
            credentials {
                username project.repoUser
                password project.repoPassword
            }
        }
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation group: 'net.logstash.logback', name: 'logstash-logback-encoder', version: '5.2'
    implementation 'org.mongodb:mongodb-driver:3.12.11'
    testImplementation group: 'junit', name: 'junit', version: '4.12'
}
```

```groovy
repoUser = elhamy
repoPassword = MySuperPass
```

```groovy
rootProject.name = 'my-app'
```

```bash
./gradlew build
./gradlew publish # The publish command was added by the maven-publish plugin
```

In the Nexus UI

- Browse server contents → Browse → maven-snapshots
- The jar file and metadata will be there

## Maven Project - Configure with Nexus

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>java-maven-app</artifactId>
    <version>1.1.0-SNAPSHOT</version>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
            </plugins>
        </pluginManagement>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <distributionManagement>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <url>http://64.225.108.229:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.4.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>6.4</version>
        </dependency>

        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver</artifactId>
            <version>3.12.11</version>
        </dependency>
    </dependencies>

</project>
```

```xml
<settings>
  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>elhamy</username>
      <password>MySuperPass</password>
    </server>
  </servers>
</settings>
```

```bash
mvn package
mvn deploy
```

In the Nexus UI

- Browse server contents → Browse → maven-snapshots
- The jar file and metadata will be there

# Nexus REST API

- Query Nexus Repository for different information
    - which components?
    - which repositories?
    - what are the versions?
- This information is needed in your CI/CD Pipeline
- Use a tool like curl or wget to execute http requests
- Provide user and credential of a Nexus user
- Use the Nexus user with the required permissions

```bash
# Get the repositories that the user has permissions
curl -u elhamy:MySuperPass -X GET 'http://64.225.108.229:8081/service/rest/v1/repositories'
# using the same command for admin returns all repositories
curl -u admin:MySuperPass -X GET 'http://64.225.108.229:8081/service/rest/v1/repositories'

# Listing components in maven-snapshots repository
curl -u elhamy:MySuperPass -X GET 'http://64.225.108.229:8081/service/rest/v1/components?repository=maven-snapshots'
# Getting a specific component by it's ID
curl -u elhamy:MySuperPass -X GET 'http://64.225.108.229:8081/service/rest/v1/components/bWF2ZW4tc25hcHNob3RzOjhhMTk4YjVjMGJjNjEzZGM1N2ZmOGFiOWQ0ZTM5NzQ3'
```

# Blob Store

- Nexus Storage to store all the uploaded files
- In the Nexus UI:
    - Repository → Blob Stores
- Storage of binary files
- Local Storage or Cloud Storage

```bash
ssh root@Nexus-Droplet-IP
ls /opt/sonatype-work/nexus3/blobs # Lists the blobs also visible in the UI
```

## Blob Store - Type

- Type field → Storage Backend
    - File → file system-based storage
    - S3 → aws cloud-based storage

# Component vs Asset

- In nexus browse if we go to maven-snapshots
    - java-maven-app and my-app are components
    - The files inside those components are all assets (jar, pom, xml files,..)
- Components:
    - abstract
    - what we are uploading
- Assets
    - actual physical package/files
    - 1 component = 1 or more assets
- Docker
    - Docker format gives assets unique identifiers (Docker layers)
    - Docker Layers == Assets
    - E.g. 2 Docker Images → 2 Components, but share same assets

# Cleanup Policies

## Creating a Cleanup Policy

- Repository → Cleanup Policies → Create Cleanup Policy
    - Name: maven
    - Format: maven2
    - Criteria → Define what we want. We can preview the effects before creating the policy

## Attach Policy to Repository

- Repositories → maven-snapshots
    - apply the created policy in the end
    - Hit save

## Scheduled Tasks

- System → Tasks
    - There will be there the Cleanup service (this is a soft delete, it will not delete from disk. for that we need to create a task to compact the blob store)
    - This lists the scheduled tasks and when they will run
    

# Exercises

This is my solution of the exercise 6 - Artifact Repository Manager with Nexus of the [TechWorld with Nana DevOps Bootcamp](https://www.techworld-with-nana.com/devops-bootcamp)

## Exercise 1: Install Nexus on a server

- Droplet with 8GB of RAM with Java 8 installed

```bash
cd /opt
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz
ls

adduser nexus
ls -l

# Nexus user needs permissions to execute nexus executable 
# and to read/write sonatype-work folder

# change ownership recursively for nexus user and nexus group
chown -R nexus:nexus nexus-3.72.0-04
chown -R nexus:nexus sonatype-work
ls -l

vim nexus-3.72.0-04/bin/nexus.rc

# uncomment the line and add nexus user, it should look like:
# run_as_user="nexus"
# Save and quit

su - nexus
/opt/nexus-3.72.0-04/bin/nexus start
ps aux | grep nexus # check that it is running
netstat -lnpt # match the PID with previous command to find the Nexus ports
```
