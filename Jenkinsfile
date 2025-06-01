pipeline {
  agent { label 'agente-docker' }

  environment {
    DOCKER_REPO = 'docker.io/usuario/demoapp'
    TAG = 'latest'
    MAVEN_HOME = tool 'Maven 3.9.6'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/jmontalvof/proyecto-ejemplo-jenkins.git', 
            branch: 'main',
            credentialsId: 'github_credentials'
      }
    }

    stage('Build con Maven') {
      steps {
        sh "${MAVEN_HOME}/bin/mvn clean package -DskipTests"
      }
    }

    stage('Tests Postman') {
      steps {
        sh 'newman run tests/coleccion.postman_collection.json'
      }
    }

    stage('Kaniko Build') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          script {
            def authConfig = """{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "${DOCKER_USER}",
      "password": "${DOCKER_PASS}"
    }
  }
}"""
            writeFile file: 'kaniko/.docker/config.json', text: authConfig
          }

          sh '''
            docker exec kaniko /kaniko/executor \
              --dockerfile=Dockerfile \
              --context=/workspace \
              --destination=${DOCKER_REPO}:${TAG} \
              --skip-tls-verify
          '''
        }
      }
    }
  }

  post {
    success {
      echo '✅ Pipeline ejecutado con éxito.'
    }
    failure {
      echo '❌ Error en el pipeline.'
    }
  }
}
