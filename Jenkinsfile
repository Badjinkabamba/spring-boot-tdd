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
                       sh "mvn test -Dgroups=unitaires"
                    }
                    post {
                      always{
                      junit 'target/surefire-reports/*.xml'
                      jacoco execPattern: 'target/jacoco.exec'
                      }
                    }
        }

        stage('Service - IntegrationTest') {
         steps{
         sh "mvn test -Dgroups=integrations"
         }
        }

        stage('Web - IntegrationTest') {
         steps{
         sh "mvn test -Dgroups=web"
         }
        }


         stage('Docker Build and Push') {
         when { expression { false } }
                    steps {
                        withDockerRegistry([credentialsId: "my-docker-hub", url: ""]) {
                            sh 'docker build -t badjinkabamba/spring-boot-tdd:$BUILD_NUMBER .'
                            sh 'docker push badjinkabamba/spring-boot-tdd:$BUILD_NUMBER'
                        }

                    }
                }

          stage('Remove Unused docker image') {
          when { expression { false } }
            steps{
                sh "docker rmi badjinkabamba/spring-boot-tdd:$BUILD_NUMBER"
            }
           }

           stage('Kubernetes Deployment - DEV') {
           when { expression { false } }
            steps {
             withKubeConfig([credentialsId: 'kubernetes-config']) {
              sh "sed -i 's#replace#badjinkabamba/spring-boot-tdd:${BUILD_NUMBER}#g' k8s_deployment_service.yaml"
              sh "kubectl apply -f k8s_deployment_service.yaml"
              }
             }
            }
    }
}
