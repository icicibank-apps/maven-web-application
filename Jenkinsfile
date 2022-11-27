node{
  
try{

def mavenHome = tool name:'Maven 3.8.6'

echo "The job name is: ${env.JOB_NAME}"
echo "The Build number is: ${env.BUILD_NUMBER}"
echo "The node name is: ${env.NODE_NAME}"

//CheckoutCode state
stage('CheckoutCode'){
git branch: 'development', credentialsId: '710f4804-187b-48c7-8b1e-5149284196fb', url: 'https://github.com/icicibank-apps/maven-web-application.git'
}

//Build
stage('Build'){
sh "${mavenHome}/bin/mvn clean package"
}

//Execute sonarqube report
stage('ExecuteSonarQubeReport'){
sh "${mavenHome}/bin/mvn sonar:sonar"
}

//UploadArtifactsIntoNexus
stage('UploadArtifactsIntoServer'){
sh "${mavenHome}/bin/mvn deploy"
}

//Deploy application into TOmcat server
stage('DeployApp'){
sshagent(['71b71013-9b01-4ac6-afab-7e596a986f49']) {
sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.5.66:/opt/apache-tomcat-9.0.68/webapps"
}
}
  
}//try closing
catch(e){
currentBuild.result = "FAILURE"
}
finally{
sendSlackNotifications(currentBuild.result)
}

} //node closing

//Function for slack notifications

def sendSlackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    colorName = 'ORANGE'
    colorCode = '#FFA500'
  } else if (buildStatus == 'SUCCESS') {
    colorName = 'GREEN'
    colorCode = '#00FF00'
  } else {
    colorName = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
