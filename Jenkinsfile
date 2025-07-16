pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'iboussari'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  options {
    skipDefaultCheckout(true)
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'dev', url: 'https://github.com/iboussari/jenkins_devops_exams.git'
      }
    }

    stage('Build & Push cast-service') {
      steps {
        dir('cast-service') {
          sh """
            docker build -t $DOCKERHUB_REPO/cast-service:$IMAGE_TAG .
            docker push $DOCKERHUB_REPO/cast-service:$IMAGE_TAG
          """
        }
      }
    }

    stage('Build & Push movie-service') {
      steps {
        dir('movie-service') {
          sh """
            docker build -t $DOCKERHUB_REPO/movie-service:$IMAGE_TAG .
            docker push $DOCKERHUB_REPO/movie-service:$IMAGE_TAG
          """
        }
       }
      }

    stage('Deploy to DEV') {
      when {
        branch 'dev'
      }
      steps {
        sh """
          kubectl config use-context ton-context
          helm upgrade --install cast-service-dev ./helm-chart \
            --namespace dev \
            --set image.tag=$IMAGE_TAG
            
          helm upgrade --install movie-service-dev ./helm-chart \
            --namespace dev \
            --set image.tag=$IMAGE_TAG
        """
      }
    }

  }

  post {
    success {
      echo "Déploiement réussi"
    }
    failure {
      echo "Échec du pipeline"
    }
  }
 }

