pipeline {
    agent any 
    tools {
      maven 'maven home'
    }
    environment {
      DOCKER_TAG = getVersion()
    }
    stages {
        stage('SCM Checkout') { 
            steps {
                git credentialsId: 'github', 
                    url: 'https://github.com/vikas99341/K8S-pipeline.git'
            }
        }
        stage('Maven Clean Build Package') { 
            steps {
                sh "mvn clean package"
            }
        }
        stage('Build Docker Image') { 
            steps {
                sh "docker build . -t vikas24775/node-morning:${DOCKER_TAG} "
            }
        }
        stage('Push Docker Image') { 
            steps {
				withCredentials([string(credentialsId: 'docker-hub-password', variable: 'dockerhubpassword')]) {
					sh "docker login -u vikas24775 -p ${dockerhubpassword}"
				}
                sh "docker push vikas24775/node-morning:${DOCKER_TAG} "
            }
        }
        stage('Ansible SSH') { 
            steps {
                ansiblePlaybook credentialsId: 'ansible-playbook', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible_home', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}
def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
