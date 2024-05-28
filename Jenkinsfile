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

        stage('Mutation Tests - PIT') {
         steps {
         sh "mvn org.pitest:pitest-maven:mutationCoverage"
         }
         post {
         always {
         pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
         }
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

            stage('Code coverage') {
                 environment {
                 SCANNER_HOME = tool 'sonar_scanner'
                 PROJECT_KEY = "spring-boot-tdd"
                 PROJECT_NAME = "spring-boot-tdd"
                 }
                 steps {
                     withSonarQubeEnv('sonar_server') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=$PROJECT_KEY \
                            -Dsonar.projectName=$PROJECT_NAME \
                            -Dsonar.java.coveragePlugin=jacoco \
                            -Dsonar.jacoco.reportPath=target/jacoco.exec \
                            -Dsonar.java.binaries=target/classes/ '''
                     }
                 }
            }

            stage("Quality Gate") {
                 steps {
                     timeout(time: 1, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                     }
                 }
             }

    }
}
