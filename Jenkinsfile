pipeline { 

  agent any
environment {
        DOCKER_REGISTRY = 'docker.io/shehabsaeed01/devsecops' // e.g., 'docker.io' for Docker Hub
        DOCKER_IMAGE_NAME = 'shehabsaeed01/numeric-app'    // e.g., 'my-org/my-repo'
        DOCKER_IMAGE_TAG = "${env.GIT_COMMIT}"              // e.g., '1.0.0' or 'latest'
        DOCKER_CREDENTIALS_ID = 'docker-hub' // Jenkins credentials ID for Docker registry
        SNYK_TOKEN = credentials('snyk-creds') // Store token in Jenkins Credentials
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

stage('SonarQube Analysis') {
    
    
    steps {
        withSonarQubeEnv('MySonarQubeServer') {
            withCredentials([string(credentialsId: 'sonar-token', variable: 'Sonar_Token')]) {
                sh "/usr/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-app -Dsonar.login=${Sonar_Token}"
            }
        }
    }
}

stage('snyk-dependency-check') {
      steps {
        
        script {
                    try {
                        sh 'snyk auth $SNYK_TOKEN'
                        sh 'snyk test --json > snyk-report.json'
                    } catch (Exception e) {
                        echo "Snyk scan failed, but continuing..."
                    }
                }
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
              sh "sed -i 's#replace#shehabsaeed01/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh "kubectl -n prod apply -f k8s_deployment_service.yaml"
            }
        
      
      }
     }
  }
}