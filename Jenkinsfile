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
    APP_CONTAINER = 'new-api-gigi'
    COMPOSE_RUNNER_IMAGE = 'docker/compose:1.29.2'
    FILE_SYNC_IMAGE = 'busybox:1.36'
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

            docker pull ${IMAGE}:latest

            docker run --rm -i \
              -v ${DEPLOY_DIR}:${DEPLOY_DIR} \
              -w ${DEPLOY_DIR} \
              ${FILE_SYNC_IMAGE} sh -c "cat > ${COMPOSE_FILE}" < ${COMPOSE_SOURCE}

            compose() {
              if command -v docker-compose >/dev/null 2>&1 && [ -f "${DEPLOY_DIR}/${COMPOSE_FILE}" ]; then
                cd ${DEPLOY_DIR}
                docker-compose "$@"
              else
                echo "docker-compose is not usable in Jenkins agent; using ${COMPOSE_RUNNER_IMAGE}"
                docker run --rm \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  -v ${DEPLOY_DIR}:${DEPLOY_DIR} \
                  -w ${DEPLOY_DIR} \
                  -e NEW_API_ENV_FILE=${NEW_API_ENV_FILE} \
                  ${COMPOSE_RUNNER_IMAGE} "$@"
              fi
            }

            compose --version
            compose -f ${COMPOSE_FILE} up -d --remove-orphans
            docker image prune -f
          '''
        }
      }
    }

    stage('Health Check') {
      steps {
        sh '''
          timeout 120 sh -c '
            while true; do
              status=$(docker inspect -f "{{if .State.Health}}{{.State.Health.Status}}{{else}}no-healthcheck{{end}}" ${APP_CONTAINER} 2>/dev/null || true)
              if [ "$status" = "healthy" ]; then
                break
              fi
              if [ "$status" = "unhealthy" ]; then
                echo "Container ${APP_CONTAINER} is unhealthy" >&2
                exit 1
              fi
              sleep 5
            done
          '
          docker exec ${APP_CONTAINER} wget -q -O - http://127.0.0.1:3000/api/status | grep -o '"success":\\s*true'
        '''
      }
    }
  }
}
