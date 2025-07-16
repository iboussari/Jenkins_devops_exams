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

    stage('Docker Login & Build/Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh """
            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

            docker build -t \$DOCKER_USER/cast-service:${IMAGE_TAG} cast-service/
            docker push \$DOCKER_USER/cast-service:${IMAGE_TAG}

            docker build -t \$DOCKER_USER/movie-service:${IMAGE_TAG} movie-service/
            docker push \$DOCKER_USER/movie-service:${IMAGE_TAG}
          """
        }
      }
    }

    stage('Deploy to Environment') {
      steps {
        script {
          def namespaceMap = [
            'dev'     : 'dev',
            'qa'      : 'qa',
            'staging' : 'staging',
            'master'  : 'prod'
          ]

          def envNamespace = namespaceMap.get(env.BRANCH_NAME)

          if (envNamespace == null) {
            error "Branche '${env.BRANCH_NAME}' non prise en charge pour le déploiement."
          }

          if (envNamespace == 'prod') {
            input message: "Déploiement en production (#${BUILD_ID}) autorisé ?"
          }

          echo "Déploiement vers namespace '${envNamespace}' avec le tag '${IMAGE_TAG}'"

          sh """
            helm upgrade --install cast-service-${envNamespace} ./charts/cast-service \
              --namespace ${envNamespace} --set image.tag=${IMAGE_TAG}

            helm upgrade --install movie-service-${envNamespace} ./charts/movie-service \
              --namespace ${envNamespace} --set image.tag=${IMAGE_TAG}
          """
        }
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

