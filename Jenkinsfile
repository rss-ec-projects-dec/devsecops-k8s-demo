pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean install"
            }
      }

      stage('Unit Tests - JUnit and Jacoco') {
        steps {
          sh "mvn test"
        }
        post {
          always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
          }
        }
      }   
  }
}