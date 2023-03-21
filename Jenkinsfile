pipeline {
	agent any
	environment {
		DOCKERHUB_CREDENTIALS=credentials('docker')
	}
	stages {
		stage("SCM") {
			steps {
				git branch: 'main', url: 'https://github.com/abhi0172/k8spipeline.git'
				}
			}

		
		stage("Image") {
			steps {
				sh 'sudo docker build -t k8sdeploy:$BUILD_TAG .'
				sh 'sudo docker tag k8sdeploy:$BUILD_TAG abhishek0322/deployment:$BUILD_TAG'
				}
			}
				
		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
		stage('Push') {

			steps {
				sh 'sudo docker push abhishek0322/deployment:$BUILD_TAG'
			}
		}
		stage("QAT Testing") {
			steps {
				sh 'sudo docker rm -f $(sudo docker ps -a -q)'
				sh 'sudo docker run -dit -p 9001:8080  abhishek0322/deployment:$BUILD_TAG'
				}
			}
		
		stage("Approval status") {
			steps {
				script {
					 Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                echo 'userInput: ' + userInput
					}
				}	
		}
		stage("Prod Env") {
			steps {
			 sshagent(['ubuntu']) {
			    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.26.95 sudo docker rm -f $(sudo docker ps -a -q)' 
	            sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.26.95 sudo docker run  -d  -p  49151:80  abhishek0322/deployment:$BUILD_TAG"
			    sh 'scp -o StrictHostKeyChecking=no deploy.yaml ubuntu@172.31.26.95:/home/ubuntu'
			    sh 'ssh ubuntu@172.31.26.95 kubectl apply -f .'
			    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.26.95 kubectl get svc'
				}
			}
		}
	}
}
