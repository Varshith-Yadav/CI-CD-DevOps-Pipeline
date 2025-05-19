# Java-K8s CI/CD Pipeline

A complete CI/CD pipeline solution for building, testing, and deploying Java applications to Kubernetes environments.

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-1.0.0-orange)

## Overview

This project provides a full-featured Continuous Integration and Continuous Deployment (CI/CD) pipeline specifically designed for Java applications that run on Kubernetes. It automates the entire software delivery process from code commit to production deployment, with appropriate safeguards and controls at each stage.

### Key Features

- **End-to-end Automation**: Automate building, testing, and deployment processes
- **Multiple Environment Support**: Development, staging, and production environments
- **Quality Gates**: Code quality checks, unit testing, integration testing
- **Security Scanning**: Vulnerability scanning for code and containers
- **Kubernetes-native**: Designed specifically for K8s deployments with best practices
- **Observability**: Built-in monitoring, logging, and alerting
- **Customizable Workflows**: Adaptable to various Java application types (Spring Boot, Quarkus, etc.)

## Architecture

The pipeline consists of the following key components:

1. **Source Code Management**: Git-based version control
2. **CI Server**: Jenkins/GitLab CI/GitHub Actions orchestration
3. **Build System**: Maven/Gradle build automation
4. **Testing Framework**: JUnit, Mockito, integration test suites
5. **Container Registry**: Docker image storage
6. **Kubernetes Deployment**: Helm charts for application deployment
7. **Configuration Management**: Externalized configuration
8. **Monitoring Stack**: Prometheus, Grafana, logging solutions

## Prerequisites

- Kubernetes cluster (v1.20+)
- Docker (20.10+)
- Git
- Java Development Kit (JDK 11 or 17)
- Maven (3.8+) or Gradle (7+)
- CI/CD platform access (Jenkins, GitLab CI, GitHub Actions, etc.)
- Container registry access (DockerHub, ECR, GCR, etc.)

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/java-k8s-cicd.git
cd java-k8s-cicd
```

### 2. Configure CI Server

#### For Jenkins:

1. Install required plugins:
   - Kubernetes plugin
   - Docker plugin
   - JUnit plugin
   - SonarQube plugin
   - Pipeline plugin

2. Create Jenkins pipeline configuration:
   - Add `Jenkinsfile` from the templates directory
   - Configure credentials for Docker registry and Kubernetes
   - Set up webhook triggers from your Git repository

#### For GitHub Actions:

1. Copy the workflow templates from `.github/workflows` to your repository
2. Configure secrets in your GitHub repository:
   - `DOCKER_USERNAME` and `DOCKER_PASSWORD`
   - `KUBE_CONFIG` - base64 encoded kubeconfig file
   - `SONAR_TOKEN` - for code quality checks

### 3. Configure Kubernetes Components

1. Create namespace for your application:
```bash
kubectl create namespace my-java-app
```

2. Apply RBAC configurations:
```bash
kubectl apply -f k8s/rbac.yaml
```

3. Configure secrets:
```bash
kubectl apply -f k8s/secrets.yaml
```

### 4. Customize Pipeline Configuration

Edit the `pipeline-config.yaml` file to specify:

- Application name and version pattern
- Docker registry path
- Testing requirements
- Environment-specific deployment targets
- Resource requirements

## Usage

### Standard Workflow

1. **Develop**: Write code and tests locally
2. **Commit**: Push changes to feature branch
3. **CI Trigger**: Automated tests and builds run
4. **Review**: Code review and QA processes
5. **Merge**: Changes merged to main branch
6. **Release**: Automated deployment to staging
7. **Promote**: Manual approval for production deployment

### Pipeline Commands

Trigger specific pipeline actions:

```bash
# Run the full pipeline
./pipeline.sh run-full

# Run only build and test phases
./pipeline.sh build-test  

# Deploy to specific environment
./pipeline.sh deploy --env=staging
```

### Monitoring Deployments

1. Check deployment status:
```bash
kubectl get deployments -n my-java-app
```

2. View pod status:
```bash
kubectl get pods -n my-java-app
```

3. Check application logs:
```bash
kubectl logs -l app=my-java-app -n my-java-app
```

## Project Structure

```
java-k8s-cicd/
├── .github/                # GitHub Actions workflow definitions
├── ci/                     # CI configuration
│   ├── Jenkinsfile         # Jenkins pipeline script
│   └── scripts/            # Helper scripts for CI processes
├── helm/                   # Helm charts for K8s deployment
│   ├── app/                # Application chart
│   └── dependencies/       # Dependency charts
├── k8s/                    # Kubernetes manifest files
│   ├── base/               # Common K8s resources
│   └── environments/       # Environment-specific overlays
├── src/                    # Sample application source
│   ├── main/               # Application code
│   └── test/               # Test code
├── pipeline-config.yaml    # Pipeline configuration
├── pipeline.sh             # Pipeline control script
└── README.md               # This file
```

## Configuration Guide

### Environment Variables

The pipeline uses the following environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `JAVA_VERSION` | JDK version | `17` |
| `BUILD_TOOL` | Maven or Gradle | `maven` |
| `DOCKER_REGISTRY` | Container registry URL | `docker.io` |
| `NAMESPACE` | Kubernetes namespace | `default` |
| `DEPLOY_STRATEGY` | Deployment strategy | `RollingUpdate` |

### Customizing Deployment Parameters

Modify `helm/app/values.yaml` to customize:
- Number of replicas
- Resource requests/limits
- Health check configurations
- Service ports and protocols
- Autoscaling parameters

## Security Considerations

The pipeline implements several security measures:

1. **Container Scanning**: Uses Trivy to scan for vulnerabilities in Docker images
2. **Secret Management**: Kubernetes secrets for sensitive information
3. **RBAC**: Principle of least privilege for service accounts
4. **Network Policies**: Restricted communication between components
5. **Image Signing**: Optional container image signing capability

## Troubleshooting

### Common Issues

1. **Pipeline Fails at Build Stage**
   - Check Java/Maven/Gradle versions
   - Verify build dependencies availability

2. **Deployment Fails**
   - Check kubectl context
   - Verify namespace existence
   - Check for sufficient cluster resources

3. **Container Crashes**
   - Review application logs
   - Check resource limits and requests

### Getting Help

- File issues on the GitHub repository
- Check the FAQ in the docs directory
- Consult the detailed documentation at `docs/troubleshooting.md`


