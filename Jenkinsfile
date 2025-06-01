pipeline {
  agent { label 'agente-docker' }

  environment {
    MAVEN_HOME = tool 'Maven 3.9.6'
    DOCKER_REPO = 'docker.io/usuario/demoapp'
    TAG = 'latest'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/usuario/demo.git', branch: 'main'
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

    stage('Build y push con Kaniko') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          script {
            // Genera el fichero de autenticación para Kaniko
            def dockerConfig = """{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "${DOCKER_USER}",
      "password": "${DOCKER_PASS}"
    }
  }
}"""
            writeFile file: 'kaniko/config.json', text: dockerConfig

            // Ejecuta Kaniko en su contenedor
            sh """
              docker exec kaniko /kaniko/executor \
                --dockerfile=Dockerfile \
                --context=/workspace \
                --destination=${DOCKER_REPO}:${TAG} \
                --skip-tls-verify \
                --verbosity=info \
                --docker-config=kaniko
            """
          }
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
