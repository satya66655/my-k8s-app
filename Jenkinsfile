pipeline {
  agent any

  environment {
    // Git
    GIT_REPO_URL        = 'https://github.com/satya66655/my-k8s-app.git'
    GIT_CREDENTIALS_ID  = 'git-credentials'

    // Docker
    DOCKERHUB           = 'satya66655'
    DOCKER_CREDENTIALS_ID = 'dockerhub-creds'

    // AWS / EKS
    AWS_REGION          = 'us-east-1'
    AWS_CREDENTIALS_ID  = 'aws-creds'
    EKS_CLUSTER         = 'andrew-cluster'
    
    // Kubernetes namespace (optional)
    KUBE_NAMESPACE      = 'default'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: "${GIT_REPO_URL}",
            credentialsId: "${GIT_CREDENTIALS_ID}"
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def img = "${DOCKERHUB}/your-app:${env.BUILD_NUMBER}"
          sh "docker build -t ${img} ."
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(
            credentialsId: "${DOCKER_CREDENTIALS_ID}",
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh "docker push ${DOCKERHUB}/your-app:${env.BUILD_NUMBER}"
        }
      }
    }

    stage('Deploy to EKS') {
      steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: "${AWS_CREDENTIALS_ID}"
        ]]) {
          sh """
            aws eks update-kubeconfig \
              --name ${EKS_CLUSTER} \
              --region ${AWS_REGION}

            kubectl set image deployment/your-app \
              your-app=${DOCKERHUB}/your-app:${BUILD_NUMBER} \
              -n ${KUBE_NAMESPACE}
          """
        }
      }
    }
  }

  post {
    success { echo 'Deployment succeeded!' }
    failure { echo 'Pipeline failed.' }
  }
}
