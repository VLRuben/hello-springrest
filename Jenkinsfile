//JENKINSFILE								                                    //GITHUB --> DOCKER --> TERRAFORM --> ANSIBLE --> AWS

def GIT_REPO_PKG = 'ghcr.io/vlruben/hello-springrest/hellospringrestpkg'        // GHCR_PKG package repository
                                                                                //def GIT_REPO_SSH = 'git@github.com:alvarodcr/hello-terraform.git'	// GIT SSH repository
def GIT_SSH = 'ssh-github'							                                // GIT SSH credentials
def GIT_USER = 'vlruben'						                                // GIT username
def GHCR_TOKEN = 'lucatic github'						                            // ghcr.io credential (token)
def AWS_KEY_SSH = 'amazon-ssh'						                            // AWS credentials for connecting via SSH
def AWS_KEY_ROOT = 'ruben-aws-credentials'		                                // AWS credentials for creating instances
//def ANSIBLE_INV = 'ansible/aws_ec2.yml' 				                        // Ansible inventory path
//def ANSIBLE_PB = 'ansible/hello_2048.yml' 				                        // Ansible playbook path
def VERSION = "1.${BUILD_NUMBER}"					                            // TAG version with BUILD_NUMBER

pipeline {
	
    agent any 
    options {
        timestamps()
        ansiColor('xterm')
    }
	
    stages {
        
	stage('DOCKER --> BUILDING & TAGGING IMAGE') {
            steps{
		sh """
		docker-compose build
                git tag ${VERSION}
                docker tag ${GIT_REPO_PKG}:latest ${GIT_REPO_PKG}:${VERSION}
		"""
		sshagent([GIT_SSH]) {
		    sh 'git push --tags'
		}
	    }	                              
        }  
        
        stage('DOCKER --> LOGIN & PUSHING TO GHCR.IO') {
            steps{ 
		withCredentials([string(credentialsId: GHCR_TOKEN, variable: 'TOKEN_GIT')]) {
		    sh """
		    echo $TOKEN_GIT | docker login ghcr.io -u ${GIT_USER} --password-stdin
		    docker push ${GIT_REPO_PKG}:${VERSION}
		    """	
		}
            }
        }   
    
    }     
}