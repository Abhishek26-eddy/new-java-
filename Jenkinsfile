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
    }    
        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{

                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=GJdx2cP2TCDyUY3EhQKgTc']) {
                              sh 'helm datree test myapp/'
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
