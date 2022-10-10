pipeline{
    agent any 
    stages{
        stage("sonar quality check"){
        agent {
            docker {
                image 'openjdk:11'
              }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                          sh 'chmod +x gradlew'
                           sh './gradlew sonarqube \
                                -Dsonar.projectKey=sampleWeb \
                                -Dsonar.host.url=http://43.204.143.109:9000 \
                                -Dsonar.login=931d16b73f7235d269487d292e39f3c6de7262f0'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                }
            }
        }
    }
}
}