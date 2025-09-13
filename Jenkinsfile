@Library ('awr-shared-library') _

pipeline {
    agent any
    options {
  buildDiscarder logRotator(numToKeepStr: '5')
  timeout(time: 10, unit: 'MINUTES')
  disableConcurrentBuilds()
  
}

    triggers {
  githubPush()
}
    environment {
        SONARQUBE_URL = "http://43.204.110.10:9000"
        SONAR_QUBE_TOKEN = credentials('Sonar_Token')
        TOMCAT_SERVER_IP = "172.31.6.60"
    }
    tools {
        maven 'Maven-3.9.10'
    }
    stages {
        
        stage("Clean WS") {
            steps{
                cleanWs()
            }
        }
        stage('GitClone') {
            steps {
                git branch: 'main', credentialsId: 'git123', url: 'https://github.com/Awr-Tech/student-reg-webapp-declarative.git'
            }
        }
        stage('Maven Clean Package') {
            steps {
                mavenBuild ()
            }
        }
        stage('Sonar Scan') {
            steps {
                sh "mvn sonar:sonar -Dsonar.url=${SONARQUBE_URL} -Dsonar.token=${SONAR_QUBE_TOKEN}"
            }
        }
        stage('Upload War to Nexus') {
            steps {
                sh "mvn clean deploy"
            }
        }
        stage('Stop Tomcat Server') {
            steps {
                sshagent(['Tomcat_Server1']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_SERVER_IP} sudo systemctl stop tomcat
                        echo stopping the tomcat process
                        sleep 15
                    """
                }
            }
        }
        stage('Copy War File to Tomcat') {
            steps {
                sshagent(['Tomcat_Server1']) {
                    sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${TOMCAT_SERVER_IP}:/opt/tomcat/webapps/"
                }
            }
        }
        stage('Start Tomcat') {
            steps {
                sshagent(['Tomcat_Server1']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_SERVER_IP} sudo systemctl start tomcat
                        echo Starting the tomcat process
                    """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
          
        sendEmailNotifications("${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build SUCCESS",
           "Build SUCCESS. Please check the console output at ${env.BUILD_URL}",
           'abdulrz1991@gmail.com' )
        
        }
        failure {
         
        
         sendEmailNotifications( "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build FAILED",
           "Build FAILED. Please check the console output at ${env.BUILD_URL}",
           'abdulrz1991@gmail.com' )
        }
    }
}