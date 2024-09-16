pipeline {
  agent any

  options {
    timeout(time: 2, unit: 'MINUTES')
  }

  environment {
    ARTIFACT_ID = "elbuo8/webapp:${env.BUILD_NUMBER}"
  }

  stages {
    stage('Building image') {
      steps {
        sh '''
        docker build -t testapp .
        '''  
      }
    }

    stage('Vulnerability Scan - Docker') {
      steps {
        script {

            
          // Ejecutar el escaneo de vulnerabilidades y almacenar el resultado
          def scanResult = sh(script: "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity=CRITICAL,HIGH --format table 127.0.0.1:5000/mguazzardo/testapp", returnStdout: true)
          
          // Contar las vulnerabilidades críticas
          def criticalCount = scanResult.count('CRITICAL')
          // Contar las vulnerabilidades críticas
          def highCount = scanResult.count('HIGH')

          // Imprimir el resultado del escaneo
          echo "Vulnerabilidades críticas encontradas: ${criticalCount}"
          echo "Vulnerabilidades high encontradas: ${highCount}"

          // Condicionar el avance del pipeline
          if (criticalCount > 1) {
            error("Se encontraron más de 1 vulnerabilidad crítica. Abortando el pipeline.")
          }
          if (highCount > 6) {
            error("Se encontraron más de 3 vulnerabilidades high. Abortando el pipeline.")
          }
        }
      }
    }
    stage('Run tests') {
      steps {
        sh "docker run testapp npm test"
      }
    }


    stage('Deploy Image') {
      steps {
        sh '''
        docker tag testapp 127.0.0.1:5001/mguazzardo/testapp
        docker push 127.0.0.1:5001/mguazzardo/testapp   
        '''
      }
    }
  }
}
