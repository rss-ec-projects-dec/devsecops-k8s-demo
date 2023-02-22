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

//      stage('Mutation Tests - PIT') {
//        steps {
//          sh "mvn org.pitest:pitest-maven:mutationCoverage"
//        }
//        post {
//          always {
//            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
//          }
//        }
//      }

      stage('SonarQube - SAST') {
        steps {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://18.207.217.48:9000 -Dsonar.login=91522f781c984d133e4cbbadb4c2eaf7f9533d9b"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
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

      stage('Kubernetes Deployment - DEV') {
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#rupesh91609/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
      }  
  }
}