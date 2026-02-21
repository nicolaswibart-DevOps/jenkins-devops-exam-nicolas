pipeline {
  agent any

  environment {
    DOCKERHUB_USER = "nicolaswibart"
    IMAGE_NAME     = "cast-service"
    CHART_DIR      = "./charts"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Push Docker image') {
      steps {
        script {
          env.IMAGE_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        }

        withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_PASS', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh """
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
            docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} cast-service
            docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
          """
        }
      }
    }

    stage('Deploy DEV') {
      steps {
        sh """
          helm upgrade --install app ${CHART_DIR} -n dev \
            --set image.repository=${DOCKERHUB_USER}/${IMAGE_NAME} \
            --set image.tag=${IMAGE_TAG}
        """
      }
    }

    stage('Deploy QA') {
      steps {
        sh """
          helm upgrade --install app ${CHART_DIR} -n qa \
            --set image.repository=${DOCKERHUB_USER}/${IMAGE_NAME} \
            --set image.tag=${IMAGE_TAG}
        """
      }
    }

    stage('Deploy STAGING') {
      steps {
        sh """
          helm upgrade --install app ${CHART_DIR} -n staging \
            --set image.repository=${DOCKERHUB_USER}/${IMAGE_NAME} \
            --set image.tag=${IMAGE_TAG}
        """
      }
    }

    stage('Deploy PROD (manual, master only)') {
      when { branch 'master' }
      steps {
        input message: "Déployer en PROD ?", ok: "Déployer PROD"
        sh """
          helm upgrade --install app ${CHART_DIR} -n prod \
            --set image.repository=${DOCKERHUB_USER}/${IMAGE_NAME} \
            --set image.tag=${IMAGE_TAG}
        """
      }
    }
  }
}
