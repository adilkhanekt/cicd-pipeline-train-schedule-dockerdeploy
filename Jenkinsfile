pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("adilkhanekt/train_schedule_node_js")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_key') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
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
                withCredentials([sshUserPrivateKey(credentialsId: 'server_key', keyFileVariable: 'KEY', usernameVariable: 'USERNAME')]) {
                    script {
                        sh "ssh -o StrictHostKeyChecking=no -i $KEY $USERNAME@$prod_server_ip \"docker pull adilkhanekt/train_schedule_node_js:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -o StrictHostKeyChecking=no -i $KEY $USERNAME@$prod_server_ip \"docker stop train-schedule\""
                            sh "ssh -o StrictHostKeyChecking=no -i $KEY $USERNAME@$prod_server_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -o StrictHostKeyChecking=no -i $KEY $USERNAME@$prod_server_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d adilkhanekt/train_schedule_node_js:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}