cat > Jenkinsfile <<'EOF'
pipeline {
  agent any

  environment {
    DOCKERHUB_USER = "nicolaswibart"
    IMAGE_NAME     = "cast-service"   // l'image déployée par le chart
    CHART_DIR      = "./charts"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build & Push Image') {
      steps {
        script {
          def tag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"

          docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
            // build depuis le dossier cast-service
            def img = docker.build("${DOCKERHUB_USER}/${IMAGE_NAME}:${tag}", "cast-service")
            img.push()
          }

          env.IMAGE_TAG = tag
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
EOF
