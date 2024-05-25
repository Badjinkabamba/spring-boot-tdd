pipeline {
    agent any
      stages{
        stage('GitHub') {
         steps {
             git (
                          url: "https://github.com/Badjinkabamba/spring-boot-tdd.git",
                          branch: "main",
                          changelog: true,
                          poll: true
                          )
         }

        }

        stage('Build Artefact') {
            steps {
               sh "mvn clean package -DskipTests=true"
               archive 'target/*.jar' //so that they can be downloaded later
            }
        }

        stage('Unit Tests - JUnit and Jacoco') {
                    steps {
                       sh "mvn test"
                    }
                    post {
                      always{
                      junit 'target/surefire-reports/*.xml'
                      jacoco execPattern: 'target/jacoco.exec'
                      }
                    }
        }

         stage('Docker Build and Push') {
                    steps {
                        withDockerRegistry([credentialsId: "my-docker-hub", url: ""]) {
                            sh 'docker build -t badjinkabamba/spring-boot-tdd:$BUILD_NUMBER .'
                            sh 'docker push badjinkabamba/spring-boot-tdd:$BUILD_NUMBER'
                        }

                    }
                }

          stage('Remove Unused docker image') {
            steps{
                sh "docker rmi badjinkabamba/spring-boot-tdd:$BUILD_NUMBER"
            }
           }

           stage('Kubernetes Deployment - DEV') {
            steps {
             withKubeConfig([credentialsId: 'kubernetes-config']) {
              sh 'sudo chmod 777 /home/vagrant/.minikube/ca.crt'
              sh 'sudo chmod 777 /home/vagrant/.minikube/profiles/minikube/client.crt'
              sh 'sudo chmod 777 /home/vagrant/.minikube/profiles/minikube/client.key'
              sh 'chmod 777 /home/vagrant/.minikube/profiles/minikube/'
              sh "sed -i 's#replace#badjinkabamba/spring-boot-tdd:${BUILD_NUMBER}#g' k8s_deployment_service.yaml"
              sh "kubectl apply -f k8s_deployment_service.yaml"
              }
             }
            }
    }
}
