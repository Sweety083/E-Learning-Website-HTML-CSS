pipeline {
    agent any

    environment {
        // Docker Hub credentials configured in Jenkins
        DOCKER_REGISTRY = "docker.io"
        DOCKER_CREDENTIALS_ID = "dockerhub"         // Replace with your Docker Hub credentials ID
        IMAGE_NAME = "sweetyraj22/e-learning-site"

        // Kubernetes namespace
        K8S_NAMESPACE = "default"

        // GitHub credentials configured in Jenkins
        GIT_CREDENTIALS_ID = "1adc61e3-4dbc-48f9-bc77-8646123f5f2c"
        GIT_REPO = "https://github.com/Sweety083/E-Learning-Website-HTML-CSS.git"
        GIT_BRANCH = "main"
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}", credentialsId: "${GIT_CREDENTIALS_ID}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Get short commit hash for tagging
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = commitHash

                    // Build Docker image
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin ${DOCKER_REGISTRY}
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes (Blue-Green)') {
            steps {
                script {
                    // Detect current active color
                    def activeColor = sh(script: "kubectl get svc e-learning-service -n ${K8S_NAMESPACE} -o=jsonpath='{.spec.selector.version}' || echo blue", returnStdout: true).trim()
                    def newColor = activeColor == "blue" ? "green" : "blue"
                    echo "Switching traffic from ${activeColor} to ${newColor}"

                    // Apply new deployment
                    sh """
                        sed 's#IMAGE_PLACEHOLDER#${IMAGE_NAME}:${IMAGE_TAG}#' k8s/deployment-${newColor}.yaml | kubectl apply -n ${K8S_NAMESPACE} -f -
                        kubectl rollout status deployment/e-learning-site-${newColor} -n ${K8S_NAMESPACE}
                    """

                    // Update service to point to new deployment
                    sh """
                        kubectl patch svc e-learning-service -n ${K8S_NAMESPACE} -p '{"spec":{"selector":{"app":"e-learning-site","version":"${newColor}"}}}'
                    """

                    echo "✅ Traffic switched to ${newColor} deployment"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Blue-Green Deployment completed successfully!'
        }
        failure {
            echo '❌ Deployment failed. Please check the Jenkins logs.'
        }
    }
}
