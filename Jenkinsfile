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
    }
}
