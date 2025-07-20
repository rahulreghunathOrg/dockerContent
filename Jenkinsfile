pipeline {
  agent { label 'agent2-jv8' }

  environment {
    DOCKERHUB_USER = 'rahuldocker314'
    IMAGE_NAME     = 'mysite-image'
    REPO_URL       = 'https://github.com/rahulreghunathOrg/dockerContent.git'
    SWARM_MANAGER  = 'tcp://18.227.46.132:2375'
  }

  stages {
    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${IMAGE_NAME}", ".")  // Build from root directory
        }
      }
    }

    stage('Push to DockerHub') {
      steps {
        withDockerRegistry([credentialsId: 'dockerhub-creds', url: '']) {
          script {
            docker.image("${IMAGE_NAME}").push()
          }
        }
      }
    }

    stage('Deploy to Docker Swarm') {
      steps {
        sh '''
        docker -H ${SWARM_MANAGER} service rm webapp || true

        docker -H ${SWARM_MANAGER} service create \
          --name webapp \
          --replicas 3 \
          --publish 80:80 \
          --mount type=volume,source=mysite-data,target=/usr/local/apache2/htdocs \
          ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Web service deployed successfully to Docker Swarm"
    }
    failure {
      echo "❌ Deployment failed. Please review logs and swarm state."
    }
  }
}
