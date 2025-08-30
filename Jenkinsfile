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
                ssh -o StrictHostkeyChecking=no ec2-user@65.2.127.175 sudo systemctl stop tomcat
                sleep 10
                """
            }
        }
        stage("Deploy War File to tomcat") {
            sshagent(['Tomcat_Server1']) {
                sh "scp -o StrictHostkeyChecking=no target/student-reg-webapp.war ec2-user@65.2.127.175:/opt/tomcat/webapps/"
            }
        }
        stage("Start Tomcat") {
            sshagent(['Tomcat_Server1']) {
                sh """
                echo Starting the tomcat process
                ssh -o StrictHostkeyChecking=no ec2-user@65.2.127.175 sudo systemctl start tomcat
                sleep 10
                """
            }
        }
    } 
     catch (err) {
    currentBuild.result = 'FAILURE'
    throw err
} finally {
    // Send email only if build was successful
    if (currentBuild.result == 'SUCCESS') {
        emailext (
            subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                <p>Hi Team,</p>
                <p>The build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> completed successfully.</p>
                <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
            """,
            to: "riyaz.awr57565@gmail.com"
        )   
}
}
def sendEmail(String subject, String body, String recipient) {
    emailext(
        subject: subject,
        body: body,
        to:recipient,
        mimeType: 'text/html'
    )
}
}