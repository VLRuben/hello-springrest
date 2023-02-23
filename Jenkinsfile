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


/*

recoger con warning ng los resultados de ejecutar el graddlew check
esto suelta archivos.xml con los resultados del pmd


TRIVY
ejecutarlo de forma manual en el repo
ver que podemos generar informes
una vez generado, se mete al Jenkinsfile y que lo vea en el path (file system) que se genere
hacerlo en un stage aparte

*/
pipeline {
	
    agent any 
    options {
        timestamps()
        ansiColor('xterm')
    }
	
    stages {
    
	stage('GRADLE --> TESTING') {
        steps{
		    sh './gradlew test'	
	    }		
            post {
                success {
                    junit 'build/test-results/**/*.xml'
                        jacoco (
                        execPattern: 'build/jacoco/**.exec'
                        )
                }
                failure {
                    echo "\033[20mFAILED!\033[0m"
                }
	        }	 
    }

    stage ('GRADLEW CHECK') {
        steps {
            sh './gradlew check'	
            recordIssues(tools: [pmdParser(pattern: 'build/reports/pmd/*.xml')])
        }
    }   

    steps{
        sh 'trivy fs --security-checks vuln,secret,config -o ${WORKSPACE}/build/reports/trivy-report.json .'	
	    }		
            post {
                success {
                    recordIssues(tools: [
                        trivy(pattern: '${WORKSPACE}/build/reports/*.json')
                    ])      

                }
                failure {
                    echo "\033[20mFAILED!\033[0m"
                }
	        }	 
    }

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
		    docker push ${GIT_REPO_PKG}:latest
		    """	
		}
            }
        }   
        
        stage('EB --> DEPLOYING') {
            steps {
                withAWS(credentials: 'ruben-aws-credentials') {
		            dir ("eb-files"){
		                sh 'eb deploy'
                    }
		        }
	    }
    }
    
}                