pipeline { 

  agent any
environment {
        DOCKER_REGISTRY = 'docker.io/shehabsaeed01/devsecops' // e.g., 'docker.io' for Docker Hub
        DOCKER_IMAGE_NAME = 'shehabsaeed01/numeric-app'    // e.g., 'my-org/my-repo'
        DOCKER_IMAGE_TAG = "${env.GIT_COMMIT}"              // e.g., '1.0.0' or 'latest'
        DOCKER_CREDENTIALS_ID = 'docker-hub' // Jenkins credentials ID for Docker registry
}
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
      }
      stage('Unit Tests') {
            steps {
              sh "mvn test"
              
            }
        } 
      stage('Build Docker Image') {
            steps {
         withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
         sh 'printenv'
         sh 'sudo docker build -t  shehabsaeed01/numeric-app:""$GIT_COMMIT"" .'
           sh 'docker push shehabsaeed01/numeric-app:""$GIT_COMMIT""'

}
            }

 post {
        success {
            echo "Docker image built and pushed successfully with tag: ${DOCKER_IMAGE_TAG}"
        }
        failure {
            echo "Failed to build or push Docker image."
        }
    }

}
     stage('K8S Deployment - PROD') {
      steps {
           withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#${imageName}#g' k8s_PROD-deployment_service.yaml"
              sh "kubectl -n prod apply -f k8s_PROD-deployment_service.yaml"
            }
        
      
      }
     }
  }
}