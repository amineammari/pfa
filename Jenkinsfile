def pipeline(commit_id)
  agent any

  stages {
    stage('Preparation') {
      steps {
        checkout scm
        sh 'git rev-parse --short HEAD > .git/commit-id'
        script {
          commit_id = readFile('.git/commit-id').trim()
        }
      }
    }

    stage('Image Build') {
      steps {
        echo "Building..."
        sh "scp -r -i \$(minikube ssh-key) ./ docker@\$(minikube ip):~/"
        sh "minikube ssh 'docker build webapp -t \$(commit_id)'"
        echo "Build Complete"
      }
    }

    stage('Deploy') {
      steps {
        echo "Deploying to Kubernetes..."
        sh "kubectl set image deployment/richardchesterwood-k8s-fleetman-webapp-angular-release-0 webapp=\$(commit_id) --manifests/workloads.yaml"
        sh "kubectl apply -f manifests/"
        sh "kubectl get all -f manifests/"
        echo "Deployment Complete"
      }
    }
  }
