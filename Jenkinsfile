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
        git branch: 'staging', url: 'https://github.com/iboussari/jenkins_devops_exams.git'
      }
    }

    stage('Docker Login & Push Images') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

            docker build -t \$DOCKER_USER/cast-service:\$IMAGE_TAG cast-service/
            docker push \$DOCKER_USER/cast-service:\$IMAGE_TAG

            docker build -t \$DOCKER_USER/movie-service:\$IMAGE_TAG movie-service/
            docker push \$DOCKER_USER/movie-service:\$IMAGE_TAG
          """
        }
      }
    }

    stage('Deploy to STAGING') {
      
      steps {
        sh """
          helm upgrade --install cast-service-dev ./charts/cast-service \
            --namespace dev --set image.tag=$IMAGE_TAG

          helm upgrade --install movie-service-dev ./charts/movie-service \
            --namespace dev --set image.tag=$IMAGE_TAG
        """
      }
    }

    stage('Validation pour PROD') {
      when {
        branch 'master'
      }
      steps {
        input message: 'Déployer en production ?'
        sh """
          helm upgrade --install cast-service-prod ./charts/cast-service \
            --namespace prod --set image.tag=$IMAGE_TAG

          helm upgrade --install movie-service-prod ./charts/movie-service \
            --namespace prod --set image.tag=$IMAGE_TAG
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

