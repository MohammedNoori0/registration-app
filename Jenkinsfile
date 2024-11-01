pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "registeration-app"
            RELEASE = "1.0.0"
            DOCKER_USER = "mohammednoori0"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/MohammedNoori0/registration-app'
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
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }

       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        }

        stage("Build & Push Docker Image") {
        steps {
            script {
                docker.withRegistry('', DOCKER_PASS) {
                    // Build the Docker image and tag it with IMAGE_NAME
                    docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    
                    // Push the specific tag
                    docker_image.push("${IMAGE_TAG}")
                    
                    // Retag the image as 'latest' and push
                    docker_image.push('latest')
                }
            }
        }
    }

          stage("Trivy Scan"){
           steps {
	           script {
                        sh("docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mohammednoori0/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table --quiet")
		        }
	           }	
           }


           stage("Cleanup Artifacts"){
           steps {
	           script {
                        sh "docker rmi ${IMAGE_NAME}":${IMAGE_TAG}
                        sh "docker rmi ${IMAGE_NAME}:latest"
		        }
	           }	
           }
    }
}
