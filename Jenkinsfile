stage('Image Build') {
    steps {
        echo "Copying Dockerfile and index.html to Minikube..."
        
        // Copie le Dockerfile
        sh "minikube cp ${WORKSPACE}/Dockerfile /tmp/Dockerfile"
        
        // Copie le fichier index.html
        sh "minikube cp ${WORKSPACE}/index.html /tmp/index.html"
        
        echo "Building Docker image with tag fleetman-webapp:${commit_id}..."
        sh "minikube ssh 'cd /tmp && docker build -t fleetman-webapp:${commit_id} .'"
        
        echo "Build complete"
        
        // Ignore errors during cleanup
        sh "minikube ssh 'rm -f /tmp/Dockerfile /tmp/index.html' || true"
    }
}

