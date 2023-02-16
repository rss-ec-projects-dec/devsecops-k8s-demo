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

      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t rupesh91609/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push rupesh91609/numeric-app:""$GIT_COMMIT"" '
         }
        }
      }  
  }
}