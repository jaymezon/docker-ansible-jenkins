pipeline{
    agent any
    tools {
      maven 'maven-3'
    }
    environment {
      DOCKER_TAG = getVersion()
    }
    stages{
        stage('SCM'){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/jaymezon/docker-ansible-jenkins'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build Image'){
            steps{
                sh "docker build . -t jaymezon/sembeapp:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push Image'){
            steps{
                // login to dockerhub account and push image
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u jaymezon -p ${dockerHubPwd}"
                }
                
                sh "docker push jaymezon/sembeapp:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
