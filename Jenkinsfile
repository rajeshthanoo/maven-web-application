node
{
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * * ')])])
    echo "the job name is : ${env.JOB_NAME}"
    echo "the Node name is : ${env.NODE_NAME}"
    echo "the work space name is : ${env.WORKSPACE}"
    
    def mavenHome = tool name: "maven3.8.6"
    try{
    stage('stagecheckout'){
        git credentialsId: '5e45d9e4-5b4c-4fc6-b8a0-a4d3f7319db9', url: 'https://github.com/rajeshthanoo/maven-web-application.git'
    }
    stage('build'){
        sh "${mavenHome}/bin/mvn clean package"
    }
    stage('sonarqubeReport'){
        sh "${mavenHome}/bin/mvn clean sonar:sonar"
    }
    stage('deploying Artifacts'){
        sh "${mavenHome}/bin/mvn  deploy"
    }
    stage('deploy tomcat server'){
        sshagent(['0f9f80e8-1bc3-4dfd-9e42-6e9873b061e5']) {
    sh "cp -R target/maven-web-application.war /opt/tomcat9/webapps/"
        }
        }
    }
    catch(e){
        currentBuild.result = "FAILED"
        throw e
    }
    finally{
        notifyBuild(currentBuild.result)
    }
}

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
