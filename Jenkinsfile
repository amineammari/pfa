pipeline {
  agent any

  environment {
    MINIKUBE_IP = ""
    COMMIT_ID = ""
  }

  stages {
    stage('Preparation') {
      steps {
        checkout scm
        sh 'git rev-parse --short HEAD > .git/commit-id'
        script {
          COMMIT_ID = readFile('.git/commit-id').trim()
          MINIKUBE_IP = sh(script: "minikube ip", returnStdout: true).trim()
        }
      }
    }

    stage('Image Build') {
      steps {
        echo "Building Docker image with tag: ${COMMIT_ID}"
        sh """
          scp -r -i \$(minikube ssh-key) ./ docker@${MINIKUBE_IP}:~/
          minikube ssh -- "docker build ~/webapp -t webapp:${COMMIT_ID}"
        """
        echo "Build Complete"
      }
    }

    stage('Deploy') {
      steps {
        echo "Deploying to Kubernetes..."
        sh "kubectl set image deployment/webapp webapp=webapp:${COMMIT_ID} || true"
        sh "kubectl apply -f manifests/"
        sh "kubectl get all -f manifests/"
        echo "Deployment Complete"
      }
    }
  }
}

