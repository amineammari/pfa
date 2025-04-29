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

        stage('Debug Path') {
            steps {
                echo 'Displaying the workspace structure for path debugging...'
                sh 'pwd' // Affiche le répertoire actuel
                sh 'ls -R' // Affiche la structure complète du projet dans le workspace
            }
        }

        stage('Image Build') {
            steps {
                echo "Setting Docker env to point to Minikube's Docker daemon..."
                sh 'eval $(minikube docker-env)'

                // Recherche dynamique du chemin du projet Angular
                script {
                    def angular_path = sh(script: "find . -type d -name 'k8s-fleetman-webapp-angular' | head -n 1", returnStdout: true).trim()
                    echo "Found Angular project at: ${angular_path}"

                    // Si le chemin n'est pas trouvé, ajouter une recherche avec un joker
                    if (angular_path == "") {
                        angular_path = sh(script: "find . -type d -name 'k8s-fleetman-webapp-angular*' | head -n 1", returnStdout: true).trim()
                        echo "Trying with wildcard: ${angular_path}"
                    }

                    // Vérifier si le chemin est toujours vide
                    if (angular_path == "") {
                        error "Angular project not found in the workspace"
                    }
                    
                    // Exécuter le build Docker avec le chemin trouvé
                    sh "docker build -t fleetman-webapp:${commit_id} ${angular_path}"
                }

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

