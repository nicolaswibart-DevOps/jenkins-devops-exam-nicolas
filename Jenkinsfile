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

        // DOCKER_HUB_PASS est un "Secret text" => on utilise string()
        withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DH_TOKEN')]) {
          sh """
            echo "\$DH_TOKEN" | docker login -u "${DOCKERHUB_USER}" --password-stdin
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
