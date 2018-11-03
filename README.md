# CICD_pipeline
Jenkins CICD with nexus repository 
This is a document regarding the CICD project of the basic “hello world” maven project



A Step by step CICD for maven project : 

1. Set Up base OS
a. Install Ubuntu base OS
2. Set up Jenkins
a. Download Jenkins from http://mirrors.jenkins.io/war-stable/latest/jenkins.war
b. Download and install java8
i. Download the java8 tar file
ii. Extract it to any directory
c. Download maven
i. Download Maven 3.3.9
ii. Extract it to any directory
d. Set environment variable for Maven and Java
i. Put that path into the /etc/profile for system variable as shown in below image

e. Download Jenkins war file
f. Start Jenkins
i. Java -jar Jenkins.war --httpPort=7070
ii. Go to localhost:7070 in the browser
iii. After that, it will ask for admin password which you can find in “~/.jenkins/secrets/initialAdminPassword” File
iv. After setting up it will ask for two option then hit suggested plugins and hit next as shown in below image

( It will take some time to install all plugins )
v. Then set your own username and password for Jenkins login
(Note: You can follow steps from https://www.edureka.co/blog/install-jenkins/ to install Jenkins )



3. Install Nexus repository for artifacts
a. Download from: https://www.sonatype.com/download-oss-sonatype
b. Then unzip tar file into any location
c. Then go to “/nexus-3.13.0-01/bin”
d. Run nexus repo with “nexus run” command
e. Default username will be admin default password will be admin123
f. Hit in the browser : localhost:8081
g. After sign in go to Settings which is found on the top
h. Then go to repositories/ maven-releases: http://localhost:8081/#admin/repository/repositories:maven-releases
i. Scroll down and make deployment policy to: “Allow redeploy” as shown in below image

j. Hit on save button
     ( Everything is set up now. It is time to build pipeline )

4. Now in Jenkins, you have to first install a new plugin for nexus repository upload
a. Open localhost:7070 in the browser
b. Login into Jenkins
c. Go to manage Jenkins in the left side.
d. Then click on manage plugins
e. Go to the available plugins tab.
f. Search for “Nexus Artifact Uploader”
g. And install it.
h. That's it!
5. Now, we will create two pipelines for build java project from maven and upload it to Nexus repository
a. Click into new projects on the left side
b. Select the pipeline and enter pipeline name
c. Add pipeline code into “pipeline” section

(Note: for daily builds and releases build I am attaching code with this documents with comments)


(In this doc for git hello world project I used my git repo)











For Daily Build:
	node {  
		git 'https://github.com/yvbathia/Hello-world.git'
        		stage('Preparation') { 
             		git branch: 'master'  // For daily builds it will be always master
              		git 'https://github.com/yvbathia/Hello-world.git'  // this is my sample hello world maven project
         		 }
          		stage('Build') {
              		// for  Running the maven build              
              		// Checking that is it Unix based or not
              		// If it is Unix based then we will go ahead
              		if (isUnix()) {
                  			sh "'mvn' package"
              		} 
         		 }

		// Following will be code for uploading the build to Nexus repository
		// it will upload to maven-releases repo and name as master
        		nexusArtifactUploader artifacts: [[artifactId: '1', classifier: '', file: 'target/my-app-1.0-SNAPSHOT.jar', type: 'jar']], credentialsId: '40364ae5-0277-48e5-856f-d8f4b01867aa', groupId: 'my-app', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: ''master''
         
   	}




















For Release Build:

(Note: It will check for parameters  if given then it will run for that branch otherwise it will build for 1.0 then 2.0 )
(If you want to enable parameters then follow steps after pipeline code)

node {
	 git 'https://github.com/yvbathia/Hello-world.git'
   if(params){
        		print(params.version)
         		stage(params.version) {
            		git branch: params.version
            		git 'https://github.com/yvbathia/Hello-world.git'
        		}
        	    stage('Build') {
          		// Checking that is it Unix based or not
          		// If it is Unix based then we will go ahead
          		if (isUnix()) {
              		sh "'mvn' package"
          		} 
       	    }
       		// Upload to nexus repository
        		nexusArtifactUploader artifacts: [[artifactId: '1', classifier: '', file: 'target/my-app-1.0-SNAPSHOT.jar', type: 'jar']], credentialsId: '40364ae5-0277-48e5-856f-d8f4b01867aa', groupId: 'my-app', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: params.version

   	}else{
       		stage('1.0') {
            		git branch: '1.0'
            		git 'https://github.com/yvbathia/Hello-world.git'
        		}
        		stage('Build') {
          		// Checking that is it Unix based or not
          		// If it is Unix based then we will go ahead
          if (isUnix()) {
              sh "'mvn' package"
          } 
       		}
       		// Upload to nexus repository
        nexusArtifactUploader artifacts: [[artifactId: '1', classifier: '', file: 'target/my-app-1.0-SNAPSHOT.jar', type: 'jar']], credentialsId: '40364ae5-0277-48e5-856f-d8f4b01867aa', groupId: 'my-app', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: 'master'
        		stage('2.0') { 
              git branch: '2.0'
              git 'https://github.com/yvbathia/Hello-world.git'
          }
          		stage('Build') {
              		// Run the maven build
              
              // Checking that is it Unix based or not
              // If it is Unix based then we will go ahead
              if (isUnix()) {
                  sh "'mvn' package"
              	} 
          		}
        nexusArtifactUploader artifacts: [[artifactId: '1', classifier: '', file: 'target/my-app-1.0-SNAPSHOT.jar', type: 'jar']], credentialsId: '40364ae5-0277-48e5-856f-d8f4b01867aa', groupId: 'my-app', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: '1.0'
       
   	}
   
}



How to enable parameters in the pipeline:

Go to build and then hit Configure pipeline
In the general section, you will see a checkbox for parameters build as shown in below image.
Add parameter type of string

