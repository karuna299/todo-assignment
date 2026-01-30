pipeline {
  agent any

  environment {
    // CHANGE THIS
    DOCKERHUB_USER = "calix114"

    // Image names
    BACKEND_IMAGE  = "${DOCKERHUB_USER}/todo-backend"
    FRONTEND_IMAGE = "${DOCKERHUB_USER}/todo-frontend"

    // Short commit hash for tagging
    IMAGE_TAG = "${env.GIT_COMMIT.take(7)}"
  }

  stages {

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Build Backend (Maven)') {
      steps {
        dir('Backend/todo-summary-assistant') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        sh """
          docker build -t ${BACKEND_IMAGE}:${IMAGE_TAG} Backend/todo-summary-assistant
          docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} Frontend/todo
        """
      }
    }

    stage('Push Docker Images') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'docker-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
          )
        ]) {
          sh """
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${BACKEND_IMAGE}:${IMAGE_TAG}
            docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}
          """
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}
