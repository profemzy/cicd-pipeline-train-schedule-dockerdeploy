pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            sh "docker.build -t train-schedule:latest 686233958969.dkr.ecr.eu-west-1.amazonaws.com/train-schedule:latest"
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }

             docker.withRegistry('https://686233958969.dkr.ecr.eu-west-1.amazonaws.com', 'aws_ecr_login') {
                        sh "docker push 686233958969.dkr.ecr.eu-west-1.amazonaws.com/train-schedule:latest"
                    }
              }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull 686233958969.dkr.ecr.eu-west-1.amazonaws.com/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d 686233958969.dkr.ecr.eu-west-1.amazonaws.com/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
