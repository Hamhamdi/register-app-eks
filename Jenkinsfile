pipeline {
    agent { label 'Jenkins-Agent' }
    tools { 
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app-eks"
        RELEASE = "1.0.0"
        AWS_ACCOUNT_ID = "694580352301" 
        AWS_REGION = "eu-north-1" 
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME = "${ECR_REPO}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        AWS_CRED = 'aws-ecr'  
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
        

        stage("Build & Tag Docker Image") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: AWS_CRED, usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        
                        sh """
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        export AWS_REGION=${AWS_REGION}
                        
                        # Login to AWS ECR
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
        
                        # Build Docker image
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
        
                        # Tag as latest
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }            

        stage("Trivy Scan Docker Image") {
            steps {
                script {
                    sh """
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG} \
                        --no-progress \
                        --scanners vuln \
                        --exit-code 0 \
                        --severity HIGH,CRITICAL \
                        --format table
                    """
                }
            }
        }

        stage("Push Docker Image to ECR") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: AWS_CRED, usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        
                        sh """
                        export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                        export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                        export AWS_REGION=${AWS_REGION}
        
                        # Push image to AWS ECR
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
        
                        # Logout
                        docker logout ${ECR_REPO}
                        """
                    }
                }
            }
        }



       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }



        
    }
}
