pipeline {
  agent any

  environment {
    DOCKERHUB_REPO = 'iboussari/ismael-boussari-datascientest-jenkins_devops_exams'
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

    stage('Build & Push Docker Image') {
      steps {
        sh """
          docker build -t $DOCKERHUB_REPO:$IMAGE_TAG .
          docker push $DOCKERHUB_REPO:$IMAGE_TAG
        """
      }
    }

    stage('Deploy to DEV') {
      when {
        branch 'dev'
      }
      steps {
        sh """
          kubectl config use-context ton-context
          helm upgrade --install jenkins_devops_exams-dev ./helm-chart \
            --namespace dev \
            --set image.tag=$IMAGE_TAG
        """
      }
    }

    stage('Deploy to QA') {
      when {
        branch 'qa'
      }
      steps {
        sh """
          helm upgrade --install jenkins_devops_exams-qa ./helm-chart \
            --namespace qa \
            --set image.tag=$IMAGE_TAG
        """
      }
    }

    stage('Deploy to Staging') {
      when {
        branch 'staging'
      }
      steps {
        sh """
          helm upgrade --install jenkins_devops_exams-staging ./helm-chart \
            --namespace staging \
            --set image.tag=$IMAGE_TAG
        """
      }
    }

    stage('Validation pour Production') {
      when {
        branch 'master'
      }
      steps {
        input message: 'Confirmez le déploiement en production ?'
        sh """
          helm upgrade --install jenkins_devops_exams-prod ./helm-chart \
            --namespace prod \
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
