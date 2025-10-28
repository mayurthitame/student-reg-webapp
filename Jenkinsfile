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
            sendEmail(currentBuild.currentResult,"mayurthitame@gmail.com")
            sendSlack(currentBuild.currentResult,"#nodejs-app")
            cleanWs()
        }    
    }
}

def sendEmail(String buildStatus,String recipient) {   
   def subject = "${buildStatus}: Job ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
   def body = """<html>
                 <head>
                    <style>
                        body {
                            font-family: 'Segoe UI', Arial, sans-serif;
                            background-color: #f4f4f4;
                            margin: 0;
                            padding: 20px;
                        }
                        .container {
                            background: #fff;
                            border-radius: 8px;
                            padding: 20px;
                            box-shadow: 0 2px 6px rgba(0,0,0,0.1);
                        }
                        .header {
                            background-color: ${buildStatus == 'SUCCESS' ? '#4CAF50' : '#F44336'};
                            color: white;
                            padding: 15px;
                            border-radius: 8px 8px 0 0;
                            text-align: center;
                            font-size: 20px;
                            font-weight: bold;
                        }
                        .content {
                            padding: 20px;
                            font-size: 14px;
                            color: #333;
                        }
                        .footer {
                            font-size: 12px;
                            color: #888;
                            text-align: center;
                            padding-top: 15px;
                            border-top: 1px solid #ddd;
                        }
                        a.button {
                            display: inline-block;
                            padding: 10px 16px;
                            background-color: ${buildStatus == 'SUCCESS' ? '#4CAF50' : '#F44336'};
                            color: white;
                            border-radius: 4px;
                            text-decoration: none;
                            margin-top: 15px;
                        }
                        a.button:hover {
                            background-color: #333;
                        }
                    </style>
                </head>
                <body>
                    <div class="container">
                        <div class="header">${env.JOB_NAME} – Build #${env.BUILD_NUMBER} ${buildStatus}</div>
                        <div class="content">
                            <p>Please review the console output for details:</p>
                            <p style="text-align:center;">
                                <a class="button" href="${env.BUILD_URL}console">View Console Output</a>
                            </p>
                        </div>
                        <div class="footer">
                            Jenkins CI • ${new Date().format("EEE, MMM d yyyy HH:mm:ss z")}
                        </div>
                    </div>
                </body>
                </html>
                """
    emailext (
        subject: subject,
        body: body,
        mimeType: 'text/html',
        to: recipient
    )   
}

def sendSlack(String buildStatus,String channel) {
    def color = buildStatus == 'SUCCESS' ? 'good' : (buildStatus == 'UNSTABLE' ? 'warning' : 'danger')
    def message = "*${buildStatus}*: Job `${env.JOB_NAME}` - Build #${env.BUILD_NUMBER} \n" +
                  "Details: ${env.BUILD_URL}console"
    slackSend(channel: channel, color: color, message: message)
}