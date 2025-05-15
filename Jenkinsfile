pipeline {
    agent any

    environment {
        // Jenkins global config keys or credentials IDs
        SONARQUBE_SERVER = 'sonarqube'
        SONAR_PROJECT_KEY = 'java-k8s-app'
        SONAR_TOKEN = credentials('sonar-token')

        IMAGE_NAME = 'ci-cd-java'
        DOCKER_IMAGE = 'varshithyadav/java-cicd-demo:latest'
        DOCKER_USER = 'varshithyadav'
        DOCKER_PASS = credentials('docker-creds')

        EMAIL_RECIPIENTS = 'varshithyadav587@gmail.com'

        // Path to your Kubernetes manifests
        K8S_MANIFEST_PATH = 'k8s/deployment.yaml'
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
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
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
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${IMAGE_NAME} ${DOCKER_IMAGE}
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    // Replace image name in manifest with latest tag
                    sh """
                        sed -i 's|image: .*|image: ${DOCKER_IMAGE}|' ${K8S_MANIFEST_PATH}
                        kubectl apply -f ${K8S_MANIFEST_PATH}
                        kubectl rollout status deployment/java-cicd-demo
                        kubectl get pods -l app=java-cicd-demo
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Optional: Validate that the service is up via curl or logs
                    sh 'kubectl logs -l app=java-cicd-demo --tail=10 || true'
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p><strong>Build Succeeded</strong></p>
                         <p>Job: ${env.JOB_NAME}</p>
                         <p>Build: ${env.BUILD_NUMBER}</p>
                         <p><a href="${env.BUILD_URL}">View Build</a></p>""",
                to: "${EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }

        failure {
            emailext(
                subject: "❌ FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p><strong>Build Failed</strong></p>
                         <p>Job: ${env.JOB_NAME}</p>
                         <p>Build: ${env.BUILD_NUMBER}</p>
                         <p><a href="${env.BUILD_URL}">View Build</a></p>""",
                to: "${EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }

        always {
            echo '⚙️ Pipeline complete.'
        }
    }
}
