node {
    def mavenHome = tool name: 'Maven-3.9.10', type: 'maven'
    stage("Git clone") {
        git branch: 'main', credentialsId: 'git123', url: 'https://github.com/Awr-Tech/student-reg-webapp-pipe.git'
    }
    stage("Maven Build") {
        sh "${mavenHome}/bin/mvn clean verify sonar:sonar"
    }
    stage("Maven Build") {
        sh "${mavenHome}/bin/mvn clean deploy"
    }
    stage("Stop Tomcat Service") {
    sshagent(['Tomcat_Server1']) {
    sh """
    echo stopping the tomcat process
    ssh -o StrictHostkeyChecking=no ec2-user@65.2.82.158 sudo systemctl stop tomcat
    sleep 10
    """
    }
 }
    stage("Deploy War File to tomcat") {
    sshagent(['Tomcat_Server1']) {
    sh "scp -o StrictHostkeyChecking=no target/student-reg-webapp.war ec2-user@65.2.82.158:/opt/tomcat/webapps/"
    }
 }
      stage("Start Tomcat") {
    sshagent(['Tomcat_Server1']) {
    sh """
    echo Starting the tomcat process
    ssh -o StrictHostkeyChecking=no ec2-user@65.2.82.158 sudo systemctl start tomcat
    sleep 10
    """
    }
 }

    
}


