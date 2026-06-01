pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  environment {
    REGISTRY = '192.168.16.102:8082'
    IMAGE_REPO = 'gigi-docker/new-api-gigi'
    IMAGE = "${REGISTRY}/${IMAGE_REPO}"
    IMAGE_TAG = "new-api-gigi-${BUILD_NUMBER}"
    DEPLOY_DIR = '/data/new-api-gigi'
    COMPOSE_SOURCE = 'deploy/docker-compose.prod.yml'
    COMPOSE_FILE = 'docker-compose.yml'
    NEW_API_ENV_FILE = '.env'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        sh '''
          docker build \
            -t ${IMAGE}:${IMAGE_TAG} \
            -t ${IMAGE}:latest \
            .
        '''
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-registry-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login ${REGISTRY} -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE}:${IMAGE_TAG}
            docker push ${IMAGE}:latest
          '''
        }
      }
    }

    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-registry-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login ${REGISTRY} -u "$DOCKER_USER" --password-stdin
            mkdir -p ${DEPLOY_DIR}
            cp ${COMPOSE_SOURCE} ${DEPLOY_DIR}/${COMPOSE_FILE}
            cd ${DEPLOY_DIR}

            docker-compose --version
            docker-compose -f ${COMPOSE_FILE} pull
            docker-compose -f ${COMPOSE_FILE} up -d
            docker image prune -f
          '''
        }
      }
    }

    stage('Health Check') {
      steps {
        sh '''
          sleep 10
          curl -fsS http://127.0.0.1:3000/api/status
        '''
      }
    }
  }
}
