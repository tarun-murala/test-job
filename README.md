Jenkins Test Job

```
pipeline {
    agent any
    tools { 
        maven 'Jenkins Maven' 
    }

    stages {
        stage('Build') {
            steps {
                snDevOpsStep()
                sh '''
                    export M2_HOME=/opt/apache-maven-3.6.0 # your Mavan home path
                    export PATH=$PATH:$M2_HOME/bin
                    mvn --version
                '''
                sh 'mvn compile'   
            }
        }

        stage('Test') {
            steps {
                snDevOpsStep()
                sh '''
                    export M2_HOME=/opt/apache-maven-3.6.0 # your Mavan home path
                    export PATH=$PATH:$M2_HOME/bin
                    mvn --version
                '''

                sh 'mvn verify'
                sh 'mvn package'

                script {
                    sshPublisher(continueOnError: false, failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName:'CorpSite UAT',
                            verbose: true,
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'target/globex-web.war',
                                    removePrefix: 'target/',
                                    remoteDirectory: '/opt/tomcat/webapps'
                                )
                            ]
                        )
                    ])
                }
                sh 'mvn compile'
                sh 'mvn verify'
                getCurrentBuildFailedTests("Test")
            }
            post {
                success {
                    junit '**/target/surefire-reports/*.xml' 
                }
            }
        }

        stage('Publish') {
            steps {
                snDevOpsStep()
                snDevOpsChange()
                snDevOpsArtifact(artifactsPayload:"""{"artifacts": [{"name": "globex-web.war","version":"3.${env.BUILD_NUMBER}.0","semanticVersion": "3.${env.BUILD_NUMBER}.0","repositoryName": "Repo1"}]}""")
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
   def response = ["curl", "-k", "-X", "POST", "-H", "Content-Type: application/json", "-d", "${json}", "https://devops.integration.user:devops@devopsoutsideintesting.service-now.com/api/sn_devops/v1/devops/tool/test?toolId=8587c1f6db538c10c93442c5059619c5&testType=Integration"].execute()
   response.waitFor()
   println response.err.text
   println response.text
 }
}
}
```
