pipeline {
  agent { label 'agent2-jv8' }

  environment {
    DOCKERHUB_USER = 'rahuldocker314'
    IMAGE_NAME     = 'mysite-image'
    FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
    SERVICE_NAME   = 'webapp'
  }

  stages {
    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${FULL_IMAGE}", ".")
        }
      }
    }

    stage('Push to DockerHub') {
      steps {
        withDockerRegistry([credentialsId: 'dockerhub-creds', url: '']) {
          script {
            docker.image("${FULL_IMAGE}").push()
          }
        }
      }
    }

    stage('Deploy to Docker Swarm (Local Agent)') {
      steps {
        sh '''
        docker info | grep "Swarm: active" || docker swarm init || true

        docker service rm webapp || true

        docker service create \
          --name webapp \
          --replicas 3 \
          --publish 80:80 \
          --mount type=volume,source=mysite-data,target=/usr/local/apache2/htdocs \
          rahuldocker314/mysite-image:latest
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Web service deployed successfully to local Docker Swarm"
    }
    failure {
      echo "❌ Deployment failed. Check Docker status and service logs."
    }
  }
}
