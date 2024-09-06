##### build the project

    mvn package

##### build Docker image called java-maven-app. Execute from root

    docker build -t java-maven-app .
    
##### push image to repo 

    docker tag java-maven-app demo-app:java-1.0