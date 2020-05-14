pipeline {
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
                sleep 5
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
