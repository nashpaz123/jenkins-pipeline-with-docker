pipeline {
    agent any 
    options {
        ansiColor('xterm')
    }
    stages {
        stage('Build and Test') {
            agent { node{
                       label "jenkins"}
            } 
            steps {
                sh 'mvn clean package'
                sh 'echo "build ran"'
                archiveArtifacts artifacts: 'gameoflife-web/target/gameoflife.war', fingerprint:true
                junit '**/target/surefire-reports/*.xml'
            }
        }
        
        stage ('Sonar Analysis') {
            agent {node{
                   label "jenkins"}
            }
            steps {
                sh 'echo "running sonar analysis"'
                sh "mvn sonar:sonar -Dsonar.host.url=http://sonarqube:9000 -Dsonar.branch=${env.BRANCH_NAME}"
            }   
        }
        
        stage ('Deploy to Integration') {
            when { branch 'master' }
            agent {node{
                   label "jenkins"}
            }
            steps {
                input message: 'Shall we deploy? (Click "Proceed" to continue)'
                build job:'../Tomcat deploy to Integration' , parameters:[string(name: 'BRANCH_NAME', value: "${env.BRANCH_NAME}")]
            }
        }
    }
}
