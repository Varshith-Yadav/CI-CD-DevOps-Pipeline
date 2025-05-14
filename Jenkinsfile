pipeline {
    agent any

    environment {
        // Update these if your names are different
        SONARQUBE_SERVER = 'MySonarQube'         // Name set in Jenkins Global Config
        SONAR_PROJECT_KEY = 'java-cicd-on-k8s'
        SONAR_TOKEN = credentials('sonar-token') // Add token via Jenkins Credentials
        IMAGE_NAME = 'java-cicd-demo:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Varshith-Yadav/CI-CD-on-k8s.git'
            }
        }

        stage('SonarQube - Code Quality') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh 'mvn clean verify sonar:sonar ' +
                       "-Dsonar.projectKey=${SONAR_PROJECT_KEY} " +
                       "-Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Trivy - Docker Image Scan') {
            steps {
                sh """
                docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy image ${IMAGE_NAME}
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Cleanup or reporting can be added here.'
        }
    }
}
