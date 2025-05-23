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
        SLACK_PROFILE_URL = 'https://fwbuzz.slack.com/team/U08N4D19SCC'
    }

    triggers {
        githubPush() // Trigger builds on push events
    }

    stages {
        stage('Init Thread') {
            steps {
                withCredentials([string(credentialsId: 'slack-bot-token', variable: 'SLACK_BOT_TOKEN')]) {
                    script {
                        def gitBranch = env.GIT_BRANCH ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                        def version = "0.0.1-${env.BUILD_ID}"
                        def repo = env.REPO

                        def mainMessage = "*Pipeline initiated by* <${env.SLACK_PROFILE_URL}|${env.INITIATED_BY}> *on* <${env.REPO}|${env.GIT_BRANCH}>.\n>*Artifact:* ${env.ARTIFACT}\n>*Version:* ${version}\n>*Repo:* <${repo}>\n>*Namespace:* ${env.NAMESPACE}\n>*Pipeline:* :stars: Visualize to troubleshoot"
                        // Send the parent message using Slack API (chat.postMessage)
                        def response = sh(
                            script: """
                                curl -s -X POST https://slack.com/api/chat.postMessage \
                                -H 'Content-type: application/json' \
                                -H 'Authorization: Bearer $SLACK_BOT_TOKEN' \
                                --data '{
                                    "channel": "${SLACK_CHANNEL}",
                                    "text": "${mainMessage.replace("\n", "\\n").replace("\"", "\\\"")}"
                                }'
                            """,
                            returnStdout: true
                        ).trim()
                        def json = readJSON text: response
                        env.SLACK_THREAD_TS = json.ts
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
                withCredentials([string(credentialsId: 'slack-bot-token', variable: 'SLACK_BOT_TOKEN')]) {
                    script {
                        def threadMessage = "Build started"
                        sh(
                            script: """
                                curl -s -X POST https://slack.com/api/chat.postMessage \
                                -H 'Content-type: application/json' \
                                -H 'Authorization: Bearer $SLACK_BOT_TOKEN' \
                                --data '{
                                    "channel": "${SLACK_CHANNEL}",
                                    "text": "${threadMessage}",
                                    "thread_ts": "${env.SLACK_THREAD_TS}"
                                }'
                            """,
                            returnStdout: true
                        ).trim()
                    }
                }
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




