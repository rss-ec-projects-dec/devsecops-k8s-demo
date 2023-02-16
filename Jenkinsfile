pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean install"
            }
      }

      stage('Unit Tests') {
        steps {
          sh "mvn test"
        }
      }   
  }
}