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
}
