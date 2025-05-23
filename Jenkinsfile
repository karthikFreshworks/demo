pipeline {
    agent {
          kubernetes {
              inheritFrom 'eks-analytics-docker-slave'
              defaultContainer 'eks-analytics-docker-slave'
              label 'eks-analytics-docker-slave'
          }
      }
    environment {
        SLACK_CHANNEL = '#freddy-insights-alerts'
        GIT_BRANCH = 'develop'
        INITIATED_BY = 'k0k079e'
        ARTIFACT = 'freddy-insights'
        VERSION = '1.0.0'
        REPO = 'https://github.com/karthikFreshworks/demo'
        NAMESPACE = 'n134370480'
        PIPELINE_TEXT = 'Visualize to troubleshoot'
        PIPELINE_ICON = ':stars:'
    }

    triggers {
        githubPush() // Trigger builds on push events
    }

    stages {
        stage('Init Thread') {
            steps {
                withCredentials([string(credentialsId: 'slack-webhook-URL', variable: 'SLACK_WEBHOOK_URL')]) {
                    script{
                        def gitBranch = env.GIT_BRANCH ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                        def version = "0.0.1-${env.BUILD_ID}"
                        def repo = env.REPO

                        def mainMessage = "*Pipeline initiated by* <span style='color:#36a3f7'><b>${env.INITIATED_BY}</b></span> *on* <span style='color:#36a3f7'><b>${env.GIT_BRANCH}</b></span>.\n>*Artifact:* freddy-insights\n>*Version:* 0.0.1-${env.BUILD_ID}\n>*Repo:* <${repo}>\n>*Namespace:* n134370480\n>*Pipeline:* :stars: Visualize to troubleshoot"

                        // Send the parent message
                        sh(
                            script: """
                                curl -s -X POST $SLACK_WEBHOOK_URL \
                                -H 'Content-type: application/json' \
                                --data '{
                                    "channel": "${SLACK_CHANNEL}",
                                    "text": "${mainMessage.replace("\n", "\\n").replace("\"", "\\\"")}"
                                }'
                            """,
                            returnStdout: true
                        ).trim()
                    }
                }
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean install -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploy stage - Add deployment steps here'
            }
        }
    }

    post {
        always {
            cleanWs() // Clean workspace after build
        }
        success {
            echo 'Build succeeded!'
            slackSend channel: "#fp-jenkins", color: "#32cd30", message: "Deployment Successful : #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", teamDomain: "fwbuzz"

        }
        failure {
            echo 'Build failed!'
        }
    }
}

