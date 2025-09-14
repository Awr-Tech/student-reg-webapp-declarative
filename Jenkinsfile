pipeline {
    agent any

    triggers {
        githubPush()
    }

    options {
        buildDiscarder logRotator(numToKeepStr: '5')
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        SONARQUBE_URL = "http://3.110.218.113:9000"
        SONAR_QUBE_TOKEN = credentials('Sonar_Token')
        TOMCAT_SERVER_IP = "172.31.6.60"
    }

    tools {
        maven 'Maven-3.9.10'
    }

    stages {
        stage("Maven Clean Package") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Sonar Scan") {
            steps {
                sh "mvn sonar:sonar -Dsonar.url=${SONARQUBE_URL} -Dsonar.token=${SONAR_QUBE_TOKEN}"
            }
        }

        stage("Upload War To Nexus") {
            steps {
                sh "mvn clean deploy"
            }
        }

        stage("Deploy To Dev Server") {
            when {
                expression { return env.BRANCH_NAME == 'development' }
            }
            steps {
                sshagent(['Tomcat_Server1']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_SERVER_IP} sudo systemctl stop tomcat
                        echo "Stopping the Tomcat Process"
                        sleep 30
                        scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${TOMCAT_SERVER_IP}:/opt/tomcat/webapps/student-reg-webapp.war
                        echo "Copying the War file to Tomcat Process"
                        ssh -o StrictHostKeyChecking=no ec2-user@${TOMCAT_SERVER_IP} sudo systemctl start tomcat
                        echo "Starting the Tomcat Process"
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
            emailext(
                subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build SUCCESS",
                body: "Build SUCCESS. Please check the console output at ${env.BUILD_URL}",
                to: 'riyaz.awr57565@gmail.com',
                mimeType: 'text/html'
            )
        }
        failure {
            emailext(
                subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build FAILED",
                body: "Build FAILED. Please check the console output at ${env.BUILD_URL}",
                to: 'riyaz.awr57565@gmail.com',
                mimeType: 'text/html'
            )
        }
    }
}