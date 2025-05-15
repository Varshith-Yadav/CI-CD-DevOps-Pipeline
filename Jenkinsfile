pipeline {
    agent any

    environment {
        // Update these if your names are different
        SONARQUBE_SERVER = 'MySonarQube'         // Name set in Jenkins Global Config
        SONAR_PROJECT_KEY = 'java-cicd-on-k8s'
        SONAR_TOKEN = credentials('sonar-token') // Add token via Jenkins Credentials
        IMAGE_NAME = 'java-cicd-demo:latest'
        DOCKER_IMAGE = 'varshithyadav/java-cicd-demo'
        DOCKER_USER = 'varshithyadav'
        DOCKER_PASS = credentials('docker-creds') // Add password via Jenkins Credentials
        EMAIL_RECIPIENTS = 'varshithyadav587@gmail.com' 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Varshith-Yadav/CI-CD-DevOps-Pipeline.git'
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

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME} $DOCKER_USER/java-cicd-demo:latest
                        docker push $DOCKER_USER/java-cicd-demo:latest
                        """
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh 'kubectl rollout status deployment/java-cicd-demo'
                    // Optional: curl the app or check logs
                }
            }
        }


    }

    post {
        success{
            emailext(
                subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>BUILD SUCCESS:</p>
                <p>Job: ${env.JOB_NAME}</p>
                <p>Build Number: ${env.BUILD_NUMBER}</p>
                <p>Build URL: ${env.BUILD_URL}</p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "${EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
        failure{
            emailext(
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>BUILD FAILED:</p>
                <p>Job: ${env.JOB_NAME}</p>
                <p>Build Number: ${env.BUILD_NUMBER}</p>
                <p>Build URL: ${env.BUILD_URL}</p>
                <p>Error: ${currentBuild.description ?: 'Unknown error'}</p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "${EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }

        always {
            echo 'Pipeline finished. Cleanup or reporting can be added here.'
        }
    }
}
