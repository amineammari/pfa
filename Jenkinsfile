def commit_id
pipeline {
    agent any
    environment {
        MINIKUBE_HOME = '/home/amine/.minikube'
        KUBECONFIG = '/home/amine/.kube/config'
    }
    stages {
        stage('Preparation') {
            steps {
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }

        stage('Image Build') {
            steps {
                echo "Setting Docker env to point to Minikube's Docker daemon..."
                sh 'eval $(minikube docker-env)'

                echo "BUILDING docker image with tag: fleetman-webapp:${commit_id} ..."
                sh "docker build -t fleetman-webapp:${commit_id} project/k8s-fleetman/microservice-source-code/release2/k8s-fleetman-webapp-angular"

                echo 'Build complete'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to Minikube'
                sh """
                    sed -i -r 's|richardchesterwood/k8s-fleetman-webapp:release2|fleetman-webapp:${commit_id}|' /var/lib/jenkins/k8s-fleetman-deploy/replicaset-webapp.yml
                """
                sh 'kubectl apply -f /var/lib/jenkins/k8s-fleetman-deploy/replicaset-webapp.yml'
                sh 'kubectl apply -f /var/lib/jenkins/k8s-fleetman-deploy/webapp-service.yml'
                sh 'kubectl get all'
                echo 'Deployment complete'
            }
        }
    }
}

