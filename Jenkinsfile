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
        REPO = 'https://github.com/karthikFreshworks/demo/tree/feature/demo'
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
                    script{
                        def gitBranch = env.GIT_BRANCH ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                        def version = "0.0.1-${env.BUILD_ID}"
                        def repo = env.REPO

                        def mainMessage = """
                        *Pipeline initiated by* `${env.INITIATED_BY}` *on* `${env.GIT_BRANCH}`.

                        *Artifact:* ${env.ARTIFACT}
                        *Version:* ${version}
                        *Repo:* <${repo}|Git Repo>
                        *Namespace:* ${env.NAMESPACE}
                        *Pipeline:* :stars: Visualize to troubleshoot
                        """

                        // Send the parent message
                        def response = sh(
                            script: """
                                curl -s -X POST https://hooks.slack.com/services/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 \
                                -H 'Content-type: application/json' \
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