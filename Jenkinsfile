pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    } 
    stages{
        // stage("sonar quality check"){
        // agent {
        //     docker {
        //         image 'openjdk:11'
        //       }
        //     }
    //       steps{
    //             script{
    //                 withSonarQubeEnv(credentialsId: 'sonar-token') {
    //                       sh 'chmod +x gradlew'
    //                        sh './gradlew sonarqube'
    //                 }

    //                 timeout(time: 1, unit: 'HOURS') {
    //                   def qg = waitForQualityGate()
    //                   if (qg.status != 'OK') {
    //                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
    //                   }
    //             }
    //         }
    //    }
    //  }
        stage("docker build & docker push"){
          steps{
            script{
                withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                        docker build -t 3.110.104.86:8083/springapp:${VERSION} . 
                        docker login -u admin -p $docker_password 3.110.104.86:8083
                        docker push 3.110.104.86:8083/springapp:${VERSION}
                        docker rmi 3.110.104.86:8083/springapp:${VERSION}
                        '''
                }
            }
          }
        } 
        stage('identifying misconfigs using datree in helm charts'){
            steps{
                script{

                    dir('kubernetes/') {
                        withEnv(['DEFAULT_TOKEN=b6d0096c-4da0-458a-bf41-a5010c6ceff6']) {
                               sh '''helm datree test myapp/'''
                       }
                    }
                }
            }
        } 
       stage("Pushing the helm charts to Nexus"){
          steps{
            script{
                withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                    dir('kubernetes/') {    
                        sh '''
                        helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                        tar -czvf  myapp-${helmversion}.tgz myapp/
                        curl -u admin:$docker_password http://3.110.104.86:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                        '''
                    }  
                }
            }
          }
        }  

        stage('Manual approval'){
            steps{
                script{
                    timeout(10) {
                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build URL and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "naveen.nmpu@gmail.com";  
                        input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
                    }
                }
            }
        }

        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigContent(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG_CONTENT')]) {
                        dir('kubernetes/') {
                          sh '''
                          helm upgrade --install --set image.repository="3.110.104.86:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/
                          '''
                        }
                    }
               }
            }
        }

        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG_CONTENT')]) {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

                     }
                }
            }
        }

    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "naveen.nmpu@gmail.com";  
		 }
	   }
}


