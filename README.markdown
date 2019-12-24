## Building the project

The project is a simple multi-module Maven project. To build the whole project, just run `mvn install` from the root directory.

## Running the game

The application is a very simple online version of [Conway's 'game of life'](http://en.wikipedia.org/wiki/Conway's_Game_of_Life). To see what the game does, run `mvn install` as described above, thengo to the gameoflife-web directory and run `mvn jetty:run`. The application will be running on http://IP_ADDRESS:9090.

## Running the acceptance tests

The acceptance tests are written using Webdriver and [Thucydides](http://thucydides.info). They are designed to run against a running server. Run the jetty instance as described about, then, in another window, go to the gameoflife-acceptance-tests directory and run `mvn clean verify`. The test reports will be generated in the `target/site/thucydides` directory.

To begin , 
## 1. clone this repository on a docker enabled linux machine.

**git clone https://github.com/nashpaz123/jenkins-pipeline-with-docker.git**

## 2. Navigate to the directory "jenkins-pipeline-with-docker" and run the below docker command to setup Jenkins, Sonarqube and Tomcat containers. (This may take up to 5-10 minutes)
Prerequisites: install docker-compose: https://docs.docker.com/compose/install/ 

e.g:

<sub>sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose</sub>

<sub>sudo chmod +x /usr/local/bin/docker-compose</sub>

**cd jenkins-pipeline-with-docker/**

**sudo docker-compose up -d**

## 3. To view all the running containers run the below command

**sudo docker-compose ps**
    
Configure the "Jenkins" instance:
## 4. Navigate to the below URL to connect to your Jenkins instance. (This may take up to 10-15 minutes)
Open http://IP_ADDRESS:8080 in a browser.

To get the initial admin password run:

**sudo docker-compose exec --user root jenkins cat /var/jenkins_home/secrets/initialAdminPassword**

Install Jenkins suggested plugins, 

<sub>(If you run into this issue: An error occurred during installation: No such plugin: cloudbees-folder ,
restart by running:</sub>

<sub>sudo docker-compose down</sub>

<sub>sudo docker-compose up -d</sub>

create the Admin user, user: 'admin' , password 'admin'.

## 5. Install maven for use in the project: (This may take up to 5 minutes)

**sudo docker-compose exec --user root jenkins apt-get update**

**sudo docker-compose exec --user root jenkins apt-get -y install maven**

## 6. Install the plugins "Deploy to container" and "Copy Artifact Plugin" 
On the top right of the screen, click Manage Jenkins → Manage plugins. From the "Available" tab ( or go to http://IP_ADDRESS:8080/pluginManager/available ), install the plugins "Deploy to container" and "Copy Artifact Plugin", they are required for the project. These plugins are used to copy the artifacts from the upstream job and deploy it to the Tomcat server.

## 7. Add 
label "jenkins" on the master server. Go to Manage Jenkins → Manage Nodes and configure the master node (or go to http://IP_ADDRESS:8080/computer/(master)/configure ). in the 'Labels' field add the text: jenkins , then save.

We add the label as we'll restrict the stages to run on the agents with the label "jenkins" in the Jenkinsfile.

# Setup Jenkins Project:

## 1. In the main Jenkins menu
click on "New item". Enter the job name as "GameofLife_pipeline". Select the "Multibranch Pipeline" as the Project type and click on "OK"

## 2. Under
Branch Sources, configure the Git repository. We will be using the repo "jenkins-pipeline-with-docker" which has already been created for the project.  Select "GIT" from the dropdown and 

enter the project repository as: https://github.com/nashpaz123/jenkins-pipeline-with-docker.git

## 3. Navigate  to jenkins and
click on "New item" and create a job called "Tomcat deploy to Integration".  Select the "Freestyle project" as the item type and click on "OK". This is the Jenkins job to deploy the built artifact on the tomcat container.

Please make sure that the name of the job matches the one mentioned in the Jenkinsfile stage 'Deploy to Integration':
https://github.com/nashpaz123/jenkins-pipeline-with-docker/blob/master/Jenkinsfile

## 4. Configure
the General section and select "This build is parameterized" and add the variable as below. Select "String Parameter" as the parameter type and enter the name of the parameter as "BRANCH_NAME" and the default value as develop. We specify the variable to copy the artifact from the correct branch.

## 5. Configure the build step
of the project to copy the artifact from the upstream project. Enter the name of the artiffact to be copied as "gameoflife-web/target/gameoflife.war"

## 6. Add 
the post-build step tp deploy to the Tomcat container and save the changes. Add the admin credentials for Tomcat and select it from the dropdown. Enter the context path and the Tomcat URL as "http://tomcat:8080"

## 7. After these jobs are created, 
you should see a build running on the master branch. If the build is not started automatically, you can manually click "Scan multipbranch pipeline Now".

## 8. View Build Results
Once the build is completed, you can navigate  to the URL "http://IP_ADDRESS:9000" to view the sonar scan results and browse the application using the URL : http://localhost:10000/gameoflife/

In Jenkins, Stage View provides extended visualization of Pipeline build history on the index page of a flow project . This represents the stages which are configured in the Jenkins Pipeline.
