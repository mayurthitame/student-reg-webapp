pipeline {  
    agent any

    environment {
        TOMCAT_IP= '13.60.65.217'
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
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}