pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "sweetyraj22/elearning-site:latest"
    KUBE_CONTEXT = "docker-desktop" // or minikube if you're using that
  }

  stages {
    stage('Clone Repo') {
  steps {
    git branch: 'main', url: 'https://github.com/Sweety083/E-Learning-Website-HTML-CSS.git'
  }
}

    stage('Build Docker Image') {
      steps {
        sh "docker build -t $DOCKER_IMAGE ."
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
          sh "docker push $DOCKER_IMAGE"
        }
      }
    }

    stage('Deploy Green to Kubernetes') {
      steps {
        sh "kubectl apply -f green-deployment.yaml --context=$KUBE_CONTEXT"
      }
    }

    stage('Switch Traffic to Green') {
      steps {
        sh "kubectl patch service elearning-service -p '{\"spec\":{\"selector\":{\"app\":\"elearning\",\"version\":\"green\"}}}' --context=$KUBE_CONTEXT"
      }
    }

    stage('Delete Blue Deployment') {
      steps {
        sh "kubectl delete deployment elearning-blue --context=$KUBE_CONTEXT"
      }
    }
  }
}
