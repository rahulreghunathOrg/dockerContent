pipeline {
  agent { label 'agent2-jv8' }

  environment {
    DOCKERHUB_USER = 'rahuldocker314'
    IMAGE_NAME     = 'mysite-image'
    FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
    DOCKER_SSH     = 'ssh://ubuntu@18.227.46.132'  // Replace with your actual user@IP
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

    stage('Deploy to Docker Swarm (via SSH)') {
      steps {
        sh """
        docker -H ${DOCKER_SSH} service rm webapp || true

        docker -H ${DOCKER_SSH} service create \
          --name webapp \
          --replicas 3 \
          --publish 80:80 \
          --mount type=volume,source=mysite-data,target=/usr/local/apache2/htdocs \
          ${FULL_IMAGE}
        """
      }
    }
  }

  post {
    success {
      echo "✅ Web service deployed successfully to Docker Swarm via SSH"
    }
    failure {
      echo "❌ Deployment failed. Please review SSH connection and swarm state."
    }
  }
}
