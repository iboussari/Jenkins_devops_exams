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

    stage('Préparation') {
      steps {
        script {
          // Détection de la branche si BRANCH_NAME est vide
          def branch = env.BRANCH_NAME ?: sh(
            script: 'git rev-parse --abbrev-ref HEAD', 
            returnStdout: true
          ).trim()

          env.BRANCH_NAME = branch
          env.IMAGE_TAG = "${env.BUILD_ID}-${env.BRANCH_NAME}"

          echo "Branche détectée : ${env.BRANCH_NAME}"
          echo "Tag d'image : ${env.IMAGE_TAG}"
        }
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

            docker build -t \$DOCKER_USER/cast-service:${env.IMAGE_TAG} cast-service/
            docker push \$DOCKER_USER/cast-service:${env.IMAGE_TAG}

            docker build -t \$DOCKER_USER/movie-service:${env.IMAGE_TAG} movie-service/
            docker push \$DOCKER_USER/movie-service:${env.IMAGE_TAG}
          """
        }
      }
    }

    stage('Déploiement selon l’environnement') {
      steps {
        script {
          def namespaceMap = [
            'dev'     : 'dev',
            'qa'      : 'qa',
            'staging' : 'staging',
            'master'  : 'prod'
          ]

          def ns = namespaceMap.get(env.BRANCH_NAME)

          if (ns == null) {
            error "Branche '${env.BRANCH_NAME}' non reconnue. Aucun déploiement effectué."
          }

          if (ns == 'prod') {
            echo "Déploiement en production désactivé automatiquement."
            echo "Image '${env.IMAGE_TAG}' est prête et poussée sur Docker Hub."
          } else {
            echo "Déploiement vers namespace '${ns}' avec image '${env.IMAGE_TAG}'"
            sh """
              helm upgrade --install cast-service-${ns} ./charts/cast-service \
                --namespace ${ns} --set image.tag=${env.IMAGE_TAG}

              helm upgrade --install movie-service-${ns} ./charts/movie-service \
                --namespace ${ns} --set image.tag=${env.IMAGE_TAG}
            """
          }
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

