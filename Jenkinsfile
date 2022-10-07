pipeline{
    agent any
    stages{
        stage("sonar quality check"){
            agent{
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sudo sh 'chmod +x gradlew'
                        sudo sh './gradlew sonarqube'
                    }
                }
            }
        }
    }
}