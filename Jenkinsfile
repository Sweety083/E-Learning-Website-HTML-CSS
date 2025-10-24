pipeline {
  agent any
  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/Sweety083/E-Learning-Website-HTML-CSS.git'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t elearning-site:latest .'
      }
    }
    stage('Push to Registry') {
      steps {
        sh 'docker tag elearning-site:latest sweetyraj22/elearning-site:latest'
        sh 'docker push sweetyraj22/elearning-site:latest'
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh 'kubectl apply -f green-deployment.yaml'
        sh 'kubectl patch service elearning-service -p \'{"spec":{"selector":{"app":"elearning","version":"green"}}}\''
      }
    }
    stage('Cleanup Blue') {
      steps {
        sh 'kubectl delete deployment elearning-blue'
      }
    }
  }
