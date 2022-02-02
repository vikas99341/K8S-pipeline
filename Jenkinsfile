pipeline {
    agent any
    tools {
      maven 'maven_home'
    }	
    environment {
      DOCKER_TAG = getVersion()
    }
    stages {
        stage('SCM Checkout') { 
            steps{
				git branch: 'main', credentialsId: 'git-credentials', 
					url: 'https://github.com/vikas99341/K8S-pipeline.git'
            }
        }
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        stage('Docker Build'){
            steps{
                sh "docker build . -t vikas24775/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('Push Docker Image') { 
            steps {
				withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerhubpassword')]) {
					sh "docker login -u vikas24775 -p ${dockerhubpassword}"
				}
                sh "docker push vikas24775/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('Ansible Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'ansible-playbook', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible_home', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
        stage('Deploy to k8s'){
            steps{
              sh "chmod +x changeTag.sh"
              sh "./changeTag.sh ${DOCKER_TAG}"
              sshagent(['ansible-playbook']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@3.83.138.229:/home/ec2-user/"
                }
		script{
			try{
				sh "sudo ssh ec2-user@3.83.138.229 kubectl apply -f ."
			}catch(error){
				 sh "sudo ssh ec2-user@3.83.138.229 kubectl create -f ."
			}
		 }
              }
        }
    }
}
def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
