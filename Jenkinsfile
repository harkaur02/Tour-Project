pipeline {
    agent any
    environment {
        IMAGE_NAME = "thethymca/html-tour-site:${BUILD_NUMBER}"
        DOCKER_REGISTRY = "https://index.docker.io/v1"
        SLACK_CHANNEL = '#jenkins-new'
    }
    tools {
        sonarQube 'SonarScanner'
    }
    stages {
        stage ('git checkout code'){
            steps {
                git branch: 'main', url: 'https://github.com/harkaur02/Tour-Project.git'
            }
        }
        stage('Docker build') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
                echo "Image built successfully!"
            }
        }
        stage ('Docker image push') {
            steps {
                echo "Image push process starting..."
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable:'DOCKER_USER', passwordVariable:'DOCKER_PASS')]){
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}
                    """
                }
            }
        }
        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'sonar-credentials', variable: 'SONAR_TOKEN')]) {
                sh 'sonar-scanner -Dsonar.login=$SONAR_TOKEN'
            }
        }
    }
    post {
        success {
            slackSend(channel: "${SLACK_CHANNEL}", message: "✅ *Pipeline Successful*: `${JOB_NAME}` build #${BUILD_NUMBER} (<${BUILD_URL}|View Build>)")
        }
        failure {
            slackSend(channel: "${SLACK_CHANNEL}", message: "🚨 *Pipeline Failed*: `${JOB_NAME}` build #${BUILD_NUMBER} (<${BUILD_URL}|View Build>)")
        }
        always {
            cleanWs()
        }
    }
}
