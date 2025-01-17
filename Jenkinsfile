import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=e8850df9-bb0c-423c-b537-4befd356eeeb',
        'AZURE_TENANT_ID=65315c02-ad62-4a35-afdd-647051a2bb85']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'Jenkins'
      def webAppName = 'Jenkins-app'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'IRG8Q~jUnotkeh6uZ9X7F.zovt_VqW9_I7XJ6b.P', usernameVariable: 'cb49ace2-c087-4fcc-890e-eb18782c8bc4')]) {
       sh '''
          az login --service-principal -u cb49ace2-c087-4fcc-890e-eb18782c8bc4 -p IRG8Q~jUnotkeh6uZ9X7F.zovt_VqW9_I7XJ6b.P -t 65315c02-ad62-4a35-afdd-647051a2bb85
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
