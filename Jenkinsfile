node { 
    def mvnTool = tool 'Maven'
    stage("checkout") {
      snDevOpsStep()
      snDevOpsChange()
      echo "Building" 
      //checkout scm
    }

    stage("build") {
      snDevOpsStep()
      snDevOpsChange()
      echo "Building" 
      sh "${mvnTool}/bin/mvn clean package"
      // artifact register - semantic version, stage name and branch are optional
      //snDevOpsArtifact(artifactsPayload:"""{"artifacts": [{"name": "tarun-artifact","version": "1.${env.BUILD_NUMBER}.0","semanticVersion": "1.${env.BUILD_NUMBER}.0","repositoryName": "maven-releases", "branchName": "master"}]}""")  
    }

    stage('test') {
      snDevOpsStep()
      snDevOpsChange()
      echo "Unit Test"
      sh "${mvnTool}/bin/mvn test"
      junit '**/target/surefire-reports/*.xml' 
      getCurrentBuildFailedTests("test")
    }

    stage("deploy") {
      snDevOpsStep()                    
      //snDevOpsPackage(name: "tarun-package-1.${env.BUILD_ID}.0", artifactsPayload: """{"artifacts": [{"name": "tarun-artifact","version": "1.${env.BUILD_NUMBER}.0","semanticVersion": "1.${env.BUILD_NUMBER}.0","repositoryName": "maven-releases", "branchName": "master"}]}""")
      snDevOpsChange()
      echo "deploy"
    }
}

def getCurrentBuildFailedTests(String stageName) {
 echo "Into getCurrentBuildFailedTests: $stageName"
 def jsonObj = [: ]
 jsonObj.put("stageName", stageName)
 def build = currentBuild.build()
 def action = build.getActions(hudson.tasks.junit.TestResultAction.class)
 if (action) {
  def result = build.getAction(hudson.tasks.junit.TestResultAction.class).getResult();
  if (result) {
   jsonObj.put("name", result.getDisplayName())
   jsonObj.put("url", result.getUrl())
   jsonObj.put("totalTests", result.getTotalCount())
   jsonObj.put("passedTests", result.getPassCount())
   jsonObj.put("failedTests", result.getFailCount())
   jsonObj.put("skippedTests", result.getSkipCount())
   jsonObj.put("duration", result.getDuration())
   jsonObj.put("buildNumber", env.BUILD_NUMBER)
   jsonObj.put("pipelineName", env.JOB_NAME)
   def json = new groovy.json.JsonBuilder(jsonObj)
   def response = ["curl", "-k", "-X", "POST", "-H", "Content-Type: application/json", "-d", "${json}", "http://devops.integration.user:devops@192.168.0.110:8080/api/sn_devops/v1/devops/tool/test?toolId=bb7526d55b3c1010598a16a0ab81c755&testType=Integration"].execute()
   response.waitFor()
   echo response.err.text
   echo response.text
 }
}
}
