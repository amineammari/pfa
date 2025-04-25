pipeline {
  agent any

  environment {
    // Récupère l’ID de commit et l’IP de Minikube
    COMMIT_ID = ""
    MINIKUBE_IP = ""
  }

  stages {
    stage('Preparation') {
      steps {
        checkout scm
        // Récupère l’ID court du commit
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
        // Copie les sources sur la VM Minikube
        sh """
          scp -i $(minikube ssh-key) -r ./ docker@${MINIKUBE_IP}:~/webapp
          minikube ssh -- "docker build ~/webapp -t webapp:${COMMIT_ID}"
        """
        echo "Build Complete"
      }
    }

    stage('Deploy') {
      steps {
        echo "Deploying to Kubernetes with image tag ${COMMIT_ID}…"
        // Met à jour l’image du déploiement et applique tes manifests
        sh "kubectl set image deployment/webapp webapp=webapp:${COMMIT_ID} --namespace default || true"
        sh "kubectl apply -f manifests/"
        sh "kubectl get all -n default"
        echo "Deployment Complete"
      }
    }
  }
}

