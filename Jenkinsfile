pipeline {
    agent any
    stages {
        
        stage('Build Started'){
            steps{
                sh "echo Building ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            }
        }
        
        stage('Installing hadolint if doesnt exist'){
            steps{
                sh '''
                    if ! [ -x "$(command -v minikube)" ]; then
                        echo 'Installing hadolint' >&2
                        make install
                    fi
                '''
            }
        }
        
        stage('Lint Docker File'){
            steps{
                sh 'make lint'
            }
        }
        
        stage('Security Scan') {
            steps { 
                aquaMicroscanner imageName: 'node:10.15.0-alpine', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
            }
        }    
        
        stage('Build Docker Image') {
            steps {
                sh 'bash run_docker.sh'
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([url: "", credentialsId: "docker_hub_id"]) {
                    sh 'bash upload_docker.sh'
                }
            }
        }
    }
}