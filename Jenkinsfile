pipeline {
  agent any

  environment {
    COMMIT_ID = ''
  }

  stages {
    stage('Preparation') {
      steps {
        checkout scm
        // Récupère l’ID court du commit
        sh 'git rev-parse --short HEAD > commit_id.txt'
        script {
          COMMIT_ID = readFile('commit_id.txt').trim()
        }
        echo "Using image tag: ${COMMIT_ID}"
      }
    }

    stage('Build & Load into Minikube') {
      steps {
        echo "Building Docker image webapp:${COMMIT_ID}…"
        sh "docker build -t webapp:${COMMIT_ID} ."
        echo "Loading image into Minikube…"
        sh "minikube image load webapp:${COMMIT_ID}"
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        echo "Updating manifest with new image tag…"
        sh """
          sed -i -E "s|(richardchesterwood/k8s-fleetman-webapp-angular:[^ ]+)|webapp:${COMMIT_ID}|g" manifests/workloads.yaml
        """
        echo "Applying manifests…"
        sh "kubectl apply -f manifests/"
        sh "kubectl get all"
      }
    }
  }

  post {
    success {
      echo "🎉 Déploiement de webapp:${COMMIT_ID} réussi !"
    }
    failure {
      echo "❌ Quelque chose a échoué, consulte les logs."
    }
  }
}

