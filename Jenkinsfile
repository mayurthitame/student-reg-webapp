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
            cleanWs()
        }
        success {
        slackSend (channel: '#student-webapp', color: "good", message: "Build - SUCCESS : ${env.JOB_NAME} #${env.BUILD_NUMBER} - URL: ${env.BUILD_URL}")
          sendEmail(
           "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build SUCCESS",
           "Build SUCCESS. Please check the console output at ${env.BUILD_URL}",
           'mayurthitame@gmail.com' )
        }
        failure {
         slackSend (channel: '#student-webapp', color: "danger", message: "Build - FAILED : ${env.JOB_NAME} #${env.BUILD_NUMBER} - URL: ${env.BUILD_URL}")
         sendEmail(
           "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build FAILED",
           "Build FAILED. Please check the console output at ${env.BUILD_URL}",
           'mayurthitame@gmail.com' )
        }
    }
}