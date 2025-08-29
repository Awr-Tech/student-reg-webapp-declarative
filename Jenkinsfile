node {
    def mavenHome = tool name: 'Maven-3.9.10', type: 'maven'
    try {
        stage("Git clone") {
            git branch: 'main', credentialsId: 'git123', url: 'https://github.com/Awr-Tech/student-reg-webapp-pipe.git'
        }
        stage("Maven Verify and SonarScan") {
            sh "${mavenHome}/bin/mvn clean verify sonar:sonar"
        }
        stage("Maven Build") {
            sh "${mavenHome}/bin/mvn clean deploy"
        }
        stage("Stop Tomcat Service") {
            sshagent(['Tomcat_Server1']) {
                sh """
                echo stopping the tomcat process
                ssh -o StrictHostkeyChecking=no ec2-user@13.201.103.6 sudo systemctl stop tomcat
                sleep 10
                """
            }
        }
        stage("Deploy War File to tomcat") {
            sshagent(['Tomcat_Server1']) {
                sh "scp -o StrictHostkeyChecking=no target/student-reg-webapp.war ec2-user@13.201.103.6:/opt/tomcat/webapps/"
            }
        }
        stage("Start Tomcat") {
            sshagent(['Tomcat_Server1']) {
                sh """
                echo Starting the tomcat process
                ssh -o StrictHostkeyChecking=no ec2-user@13.201.103.6 sudo systemctl start tomcat
                sleep 10
                """
            }
        }
    } 
     catch (err) {
        echo "An error occurred: ${e.getMessage()}"
        currentBuild.result = 'FAILURE'
    } finally {
        def buildStatus = currentBuild.result ?: 'SUCCESS'
        def colorcode = buildStatus == 'SUCCESS' ? 'good' : 'danger'
        slackSend (channel: 'lic-appteam', color: "${colorcode}", message: "Build - ${buildStatus} : ${env.JOB_NAME} #${env.BUILD_NUMBER} - URL: ${env.BUILD_URL}")
        sendEmail(
           "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build ${buildStatus}",
           "Build ${buildStatus}. Please check the console output at ${env.BUILD_URL}",
           'abdulrz1991@gmail.com' )
    }
 
}

def sendEmail(String subject, String body, String recipient) {
    emailext(
        subject: subject,
        body: body,
        to: recipient,
        mimeType: 'text/html'
    )
}