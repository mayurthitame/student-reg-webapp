@Library('JenkinsSharedLib')_
pipeline {  
    agent any

    triggers {
        githubPush()
    }

    environment {
        TOMCAT_IP= '172.31.44.101'
        SONAR_TOKEN = credentials('sonarToken')
        SONAR_URL ='http://172.31.33.201:9000/'
    }
    tools {
         maven 'Maven-3.9.11'
        }


    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Sonar Scan') {
            steps {
                sh 'mvn clean verify sonar:sonar -Dsonar.token=${SONAR_TOKEN} -Dsonar.host.url=${SONAR_URL}'
            }
        }
        stage('Upload to Nexus') {
            steps {
                sh 'mvn clean deploy'
            }
        }
        stage('deploy war file to qa server')
        {
            when {
               expression { return env.BRANCH_NAME == 'development' }
            }
            steps {
                sshagent(['TomcatServer_SSH_Credentials']) {
                    echo "deploying to QA server"
                }
            }
        }
        stage('deploy war file to Prod server')
        {
            when {
               expression { return env.BRANCH_NAME == 'main' }
            }
            steps {
                sshagent(['TomcatServer_SSH_Credentials']) {
                    echo "deploying to Prod server"
                }
            }
        }
         stage("triggering selenium tests")
        {
            steps {
                build job: 'student-reg-webapp-testing', wait: true
                // Add any additional steps needed after the test job completes
                echo "Selenium tests completed"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
              sshagent(['TomcatServer_SSH_Credentials']) {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_IP} sudo systemctl stop tomcat"
                    sh "sleep 20"
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_IP} rm /opt/tomcat/webapps/student-reg-webapp.war"
                    sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${TOMCAT_IP}:/opt/tomcat/webapps/student-reg-webapp.war"
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_IP} sudo systemctl start tomcat"  
                }
            }
        }
       
    }
     post {
        always {
            sendEmailNotification(currentBuild.currentResult,"mayurthitame@gmail.com")
            sendSlackNotification(currentBuild.currentResult,"#nodejs-app")
            cleanWs()
        }    
    }
}
