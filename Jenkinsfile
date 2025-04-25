pipeline {
  agent any

  environment {
    COMMIT_ID = ''
  }

  stages {
    stage('Preparation') {
      steps {
        checkout scm
        sh 'git rev-parse --short HEAD > commit_id.txt'
        script {
          COMMIT_ID = readFile('commit_id.txt').trim()
        }
        echo "Commit tag: ${COMMIT_ID}"
      }
    }

    stage('Install & Build Frontend') {
      steps {
        echo "Installing NPM dependencies…"
        sh 'npm install'
        echo "Building Angular app…"
        // adapte --prod si nécessaire ou la commande exact de ton projet
        sh 'npm run build -- --output-path=dist'
      }
    }

    stage('Build & Load Docker Image') {
      steps {
        echo "Building Docker image webapp:${COMMIT_ID}"
        // Attention : ton Dockerfile doit COPY dist/ (sans slash leading)
        sh "docker build -t webapp:${COMMIT_ID} ."
        echo "Loading image into Minikube"
        sh "minikube image load webapp:${COMMIT_ID}"
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        echo "Updating manifests with image tag"
        sh """
          sed -i -E "s|image: [^ ]+|image: webapp:${COMMIT_ID}|g" manifests/workloads.yaml
        """
        echo "Applying manifests…"
        sh 'kubectl apply -f manifests/'
        sh 'kubectl get all'
      }
    }
  }

  post {
    success { echo "✅ Déploiement webapp:${COMMIT_ID} terminé !" }
    failure { echo "❌ Échec du pipeline, regarde les logs !" }
  }
}

