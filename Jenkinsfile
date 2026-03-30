pipeline {
    agent any
    tools {
        jdk 'java17'
        maven 'maven'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('account_id')
        AWS_ECR_REPO_NAME = credentials('ECR_REPO')
        AWS_DEFAULT_REGION = 'us-east-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from Git"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Mahmoud123759/register-app.git'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

       stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv('sonar-server') { 
					sh """
						mvn clean verify sonar:sonar \
						  -Dsonar.projectKey=my-app \
						  -Dsonar.projectName='my-app' \
					"""
		        }
	           }	
           }
       }

       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }	
            }

        }

		stage("Docker Image Build") {
		    steps {
		        script {
		            sh 'docker system prune -f'
		            sh 'docker container prune -f'
		            sh 'docker build -t ${AWS_ECR_REPO_NAME} .'
		        }
		    }
		}
	    stage("ECR Image Pushing") {
            steps {
                script {
                        sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                        sh 'docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                        sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                }
            }
        }

       stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER} > trivyimage.txt' 
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                script {
                   
                    sh "docker rmi ${AWS_ECR_REPO_NAME}:${BUILD_NUMBER} || true"
                    
                }
            }
        }

	       
