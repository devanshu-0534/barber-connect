pipeline {
          sh 'npm test'
        }
      }
    }

    stage('Install Frontend') {
      steps {
        dir('client') {
          sh 'npm ci'
        }
      }
    }

    stage('Build Frontend') {
      steps {
        dir('client') {
          sh 'npm run build'
        }
      }
    }

    stage('Docker Build') {
      steps {
        script {
          def backendImage = "${env.REGISTRY ? env.REGISTRY + '/' : ''}${env.IMAGE_PREFIX}-backend:${env.TAG}"
          def frontendImage = "${env.REGISTRY ? env.REGISTRY + '/' : ''}${env.IMAGE_PREFIX}-frontend:${env.TAG}"

          sh "docker build -t ${backendImage} ./backend"
          sh "docker build -t ${frontendImage} ./client"

          // Optionally push only if REGISTRY is provided
          if (env.REGISTRY?.trim()) {
            docker.withRegistry("https://${env.REGISTRY}", env.DOCKER_CREDENTIALS) {
              sh "docker push ${backendImage}"
              sh "docker push ${frontendImage}"
            }
          } else {
            echo "REGISTRY not set — images built locally only. Set REGISTRY env to push."
          }
        }
      }
    }

    stage('Deploy (docker-compose)') {
      steps {
        // This assumes the Jenkins agent has docker & docker-compose and proper permissions
        sh 'docker-compose -f docker-compose.yml up -d --build'
      }
    }
  }

  post {
    always {
      echo "Pipeline finished."
    }
    success {
      echo "Build succeeded: ${env.BUILD_NUMBER}"
    }
    failure {
      echo "Build failed. Check console output."
    }
  }
}