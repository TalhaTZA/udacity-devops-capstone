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
                    if ! [ -x "$(command -v hadolint)" ]; then
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
        
        /* stage('Security Scan') {
            steps { 
                aquaMicroscanner imageName: 'node:10.15.0-alpine', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
            }
        } */   
        
        stage('Build Docker Image') {
            steps {
                sh 'bash build_docker.sh'
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withDockerRegistry([url: "", credentialsId: "docker_hub_id"]) {
                    sh 'bash upload_docker.sh'
                }
            }
        }
        
        stage('Deploying') {
              steps{
                  echo 'Deploying to AWS...'
                  withAWS(credentials: 'aws-capstone', region: 'us-west-2') {
                      sh "cd kubernetes"
                      sh "aws eks --region us-west-2 update-kubeconfig --name udacity-capstone"
                      sh "kubectl config use-context arn:aws:eks:us-west-2:363487036927:cluster/udacity-capstone"
                      sh "kubectl apply -f cluster_deploy.yaml"
                      sh "kubectl get nodes"
                      sh "kubectl get deployments"
                      sh "kubectl get pod -o wide"
                      sh "kubectl get service/capstone-app"
                }
            }
        }
        
        stage('Checking if app is up') {
              steps{
                  echo 'Checking if app is up...'
                  withAWS(credentials: 'aws-capstone', region: 'us-west-2') {
                     sh "curl https://9FDD74559E5FF6931C93E4F2F531F8B5.sk1.us-west-2.eks.amazonaws.com"
                }
            }
        }
        
        stage('Checking rollout') {
              steps{
                  echo 'Checking rollout...'
                  withAWS(credentials: 'aws-capstone', region: 'us-west-2') {
                     sh "kubectl rollout status deployments/capstone-app"
                }
            }
        }
        
        stage("Cleaning up") {
              steps{
                    echo 'Cleaning up...'
                    sh "docker system prune"
            }
        }
    }
}