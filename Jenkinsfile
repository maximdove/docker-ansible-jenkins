pipeline{
    agent any
    tools {
      maven 'maven3'
    }
    
    environment {
      DOCKER_TAG = getVersion()
    }
    
    stages{
        stage('SCM'){
            steps{
                git branch: 'main', 
                    url: 'https://github.com/maximdove/docker-ansible-jenkins.git'
            }
        }
       
       stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
       }
       
       stage('Build Docker Image'){
            steps{
                sh "docker build . -t maximdove/docker-ansible-jenkins:${DOCKER_TAG} "
            }
        }
        
        stage('Publish Docker Image'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u maximdove -p ${dockerHubPwd}"
                }
                
                sh "docker push maximdove/docker-ansible-jenkins:${DOCKER_TAG} "
            }
        }
        
        stage('Run Docker Image'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server3', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
