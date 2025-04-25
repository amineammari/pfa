pipeline {
  agent any

  environment {
    COMMIT_ID = ""
    MINIKUBE_IP = ""
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
        echo "Building Docker image: webapp:${COMMIT_ID}"
        sh """
          scp -i \$(minikube ssh-key) -r ./ docker@${MINIKUBE_IP}:~/webapp
          minikube ssh -- "docker build ~/webapp -t webapp:${COMMIT_ID}"
        """
        echo "Build Complete"
      }
    }

    stage('Deploy') {
      steps {
        echo "Deploying to Kubernetes with image tag ${COMMIT_ID}…"
        sh """
          kubectl set image deployment/webapp webapp=webapp:${COMMIT_ID} --namespace default || true
          kubectl apply -f manifests/
          kubectl get all -n default
        """
        echo "Deployment Complete"
      }
    }
  }
}

