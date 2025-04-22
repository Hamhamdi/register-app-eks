
pipeline {
    agent { label 'Jenkins-Agent' }
    tools { 
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-eks"
            RELEASE = "1.0.0"
            DOCKER_USER = "hamdihamza"
            DOCKER_PASS = 'docker-hub'
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
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Hamhamdi/register-app-eks'
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
	//stop quality gate for later
	/* stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }	
            }

        } */

	stage("Build & Push Docker Image") {
	    steps {
	        script {
	            docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
	                def docker_image = docker.build("${IMAGE_NAME}")
	                docker_image.push("${IMAGE_TAG}")
	                docker_image.push("latest")
	            }
	        }
	    }
	}

	    

    }
}
