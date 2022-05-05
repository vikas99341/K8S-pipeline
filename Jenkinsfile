pipeline {
    agent any
    tools {
		maven 'Maven home'
	}
	environment {
      DOCKER_TAG = getVersion()
    }
    stages {
        stage('SCM Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-hub-credentials', 
				url: 'https://github.com/vikas99341/K8S-pipeline.git'
            }
        }
        stage('Maven-Steps') {
            steps {
				sh 'mvn clean package '
            }
        }
        stage('Docker-Build') {
            steps {
				sh 'docker build . -t vikas24775/nodeapp:${DOCKER_TAG}'
            }
        }
        stage('Docker-Push') {
            steps {
				withCredentials([string(credentialsId: 'dockerHubPwd', variable: 'dockerHubPwd')]) {
					sh 'docker login -u vikas24775 -p ${dockerHubPwd}'
				}
			sh 'docker push vikas24775/nodeapp:${DOCKER_TAG}'
            }
        }
    }
}

def getVersion(){
    def commitHash = sh returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
