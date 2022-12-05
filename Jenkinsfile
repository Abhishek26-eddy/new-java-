pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonartoken') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }
                }  
            }
        }
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass')]) {
                             sh '''
                                docker build -t 3.12.132.208:8083/springapp:${VERSION} .
                                docker login -u admin -p $nexus_pass 3.12.132.208:8083 
                                docker push  3.12.132.208:8083/springapp:${VERSION}
                                docker rmi 3.12.132.208:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
        stage('indentifying helm version'){
            steps{
                script{
                    dir('kubernetes/') 
                        {
                            sh 'helm version'
                        }
                    }
                }
            }
        stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$docker_password http://3.12.132.208:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }
    }    
    post {
		    always {
			        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "abhishek.prajapati@octrotalk.com";  
		  }
	}    
}
