pipeline {
    agent {
          kubernetes {
              inheritFrom 'eks-analytics-docker-slave'
              defaultContainer 'eks-analytics-docker-slave'
              label 'eks-analytics-docker-slave'
          }
      }

    triggers {
        githubPush() // Trigger builds on push events
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                slackSend channel: "#fp-jenkins", message: "Deployment Initiated by ${env.BUILD_USER} on Local", teamDomain: "fwbuzz"
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