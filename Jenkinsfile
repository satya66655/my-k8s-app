pipeline {
  agent any
  environment {
    DOCKERHUB   = 'satya66655'
    EKS_CLUSTER = 'andrew-cluster'
    AWS_REGION  = 'us-east-1'
  }
  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/your-org/your-app.git',
            credentialsId: 'git-credentials'
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
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh "docker push ${DOCKERHUB}/your-app:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Deploy to EKS') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-creds']]) {
          sh """
            aws eks update-kubeconfig --name ${EKS_CLUSTER} --region ${AWS_REGION}
            kubectl set image deployment/your-app \
              your-app=${DOCKERHUB}/your-app:${BUILD_NUMBER} -n default
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
