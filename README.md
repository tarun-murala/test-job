Jenkins Test Job

```
node {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        NEXUS_REPOSITORY = "maven-releases"
        ARTIFACT_ID = "tarun-artifact"
    }
    stages {
        stage("checkout") {
            steps {
                snDevOpsStep()
                snDevOpsChange()
                echo "Building" 
                checkout scm
            }
        }

        stage("build") {
            steps {
                snDevOpsStep()
                snDevOpsChange()
                echo "Building" 
                sh "mvn clean package"
                // artifact register - semantic version, stage name and branch are optional
                snDevOpsArtifact(artifactsPayload:"""{"artifacts": [{"name": "${ARTIFACT_ID}","version": "1.${env.BUILD_NUMBER}.0","semanticVersion": "1.${env.BUILD_NUMBER}.0","repositoryName": "${NEXUS_REPOSITORY}"}]}""")  
                
            }
        }

        stage('test') {
            steps {
                snDevOpsStep()
                snDevOpsChange()
                echo "Unit Test"
                sh "mvn test"
                getCurrentBuildFailedTests("test")
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml' 
                }
          }
        }

        stage("deploy") {
            steps{
                snDevOpsStep()                    
                snDevOpsPackage(name: "tarun-package-1.${env.BUILD_ID}.0", artifactsPayload: """{"artifacts": [{"name": "${ARTIFACT_ID}","version": "1.${env.BUILD_NUMBER}.0","semanticVersion": "1.${env.BUILD_NUMBER}.0","repositoryName": "${NEXUS_REPOSITORY}"}]}""")
                snDevOpsChange()
                echo "deploy"
            }
        }
    }
}

def getCurrentBuildFailedTests(String stageName) {
 def build = currentBuild.build()
 def action = build.getActions(hudson.tasks.junit.TestResultAction.class)
 if (action) {
  def result = build.getAction(hudson.tasks.junit.TestResultAction.class).getResult();
  if (result) {
   def jsonString = "{'stageName':$stageName, 'name':${result.getDisplayName()}, 'url':${result.getUpUrl()}, 'totalTests':${totalTests}, 'passedTests':${result.getPassCount()}, 'failedTests':${result.getFailCount()}, 'skippedTests':${result.getSkipCount()}, 'duration':${result.getDuration()}, 'buildNumber':${env.BUILD_NUMBER}, 'pipelineName':${env.JOB_NAME}}"
   def parser = new JsonSlurper()
   def json = parser.parseText(str)
   def response = ["curl", "-k", "-X", "POST", "-H", "Content-Type: application/json", "-d", "${json}", "https://devops.integration.user:devops@192.168.0.110:8080/api/sn_devops/v1/devops/tool/test?toolId=bb7526d55b3c1010598a16a0ab81c755&testType=Integration"].execute()
   response.waitFor()
   println response.err.text
   println response.text
 }
}
}
```
