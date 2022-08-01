def ecrLoginHelper="docker-credential-ecr-login"
def region="ap-northeast-2"
def ecrUrl="206101987629.dkr.ecr.ap-northeast-2.amazonaws.com"
def repository="test"
def deployHost="3.36.102.239"

pipeline {
    agent any

    stages {
        stage('Pull Codes') {
            steps {
                checkout scm
            }
        }
        stage('Build Codes') {
            steps {
                sh """
                ./gradlew clean build
                """
            }
        }
        stage('Build Docker Image & Push to AWS ECR') {
            steps {
                withAWS(region:"${region}", credentials:"aws-key") {
                    ecrLogin()
                    sh """
                        curl -O https://amazon-ecr-credential-helper-releases.s3.us-east-2.amazonaws.com/0.4.0/linux-amd64/${ecrLoginHelper}
                        chmod +x ${ecrLoginHelper}
                        mv ${ecrLoginHelper} /usr/local/bin
                        ./gradlew jib -Djib.to.image=${ecrUrl}/${repository}:${currentBuild.number} -Djib.console='plain'
                    """
                }
            }
        }
        stage('Deploy to AWS EC2') {
            steps {
                sshagent(credentials : ["deploy-key"]) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${deployHost} \
                     'aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 206101987629.dkr.ecr.ap-northeast-2.amazonaws.com/${repository}; \
                      sleep 3; \
                      docker run -d -p 80:8080 -t ${ecrUrl}/${repository}:${currentBuild.number};'"
                }
            }
        }
    }
}