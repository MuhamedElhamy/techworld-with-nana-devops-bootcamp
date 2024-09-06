# Build Tools & Package Manager

What are build and package manager tools?  
After the application is developed we need to get it from the development repo to the server to be ready for production, but we will face a problem: how could we move hundreds of files to the server? So we got to use something else called artifact which is a compressed file AKA package which shows all the files we need to run the app which will be hosted on artifact repository manager because we need to deploy it multiple times (Nexus).   
Packaging happened in three stages:  
Compiling  Compressing  .tar or other format depend on the language

- for Java we have. maven and gradle
- for JS we have npm and yarn
- npm and yarn are package managers an NOT build tools like maven and gradle
    - they install dependencies but are not used for transpilling JS code
- the JS zip/tar file includes application code, but NOT the dependencies
    - To run on the server we need to install the dependencies first
    - unpack zip/tar
    - run the app
- To create a NodeJS app artifact: $ npm pack
- $ npm start → run the application
