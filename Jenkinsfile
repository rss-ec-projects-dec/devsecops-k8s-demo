pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "rupesh91609/numeric-app:${GIT_COMMIT}"
    applicationURL="http://devesceops-491993375.us-east-1.elb.amazonaws.com"
    applicationURI="/increment/99"
  }

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
              sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://3.86.183.231:9000 -Dsonar.login=91522f781c984d133e4cbbadb4c2eaf7f9533d9b"
        }
      }

//      stage('Vunlnerability Scan - Docker') {
//        steps {
//          sh "mvn dependency-check:check"
//        }
//      }

       stage('Vunlnerability Scan - Docker') {
        steps {
          parallel(
            "Dependency Scan": {
              sh "mvn dependency-check:check"
            },
            "Trivy Scan":{
              sh "bash trivy-docker-image-scan.sh"
            },
//            "OPA Conftest": {
//              sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
//            }
          )
        }
      }     

      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'sudo docker build -t rupesh91609/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push rupesh91609/numeric-app:""$GIT_COMMIT"" '
         }
        }
      }

//      stage('Vulnerability Scan - Kubernetes') {
//        steps {
//         sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
//        }
//      }      

//      stage('Kubernetes Deployment - DEV') {
//        steps {
//         withKubeConfig([credentialsId: 'kubeconfig']) {
//            sh "sed -i 's#replace#rupesh91609/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
//            sh "kubectl apply -f k8s_deployment_service.yaml"
//          }
//        }
//      }

        stage('Vulnerability Scan - Kubernetes') {
          steps {
            parallel(
              "OPA Scan"; {
                sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
              },
            "Kubesec Scan": {
              sh "bash kubesec-scan.sh"
            }
           ) 
          }
        }

        stage('Kubernetes Deployment - DEV') {
        steps {
          parallel(
            "Deployment": {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "bash k8s-deployment.sh"
              }
            },
            "Rollout Status": {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "bash k8s-deployment-rollout-status.sh"
              }
            }
          )
        }
      }    


  } 
      
  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
    }
  }

  
}