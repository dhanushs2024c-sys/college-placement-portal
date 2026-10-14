pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "dhan01/noticeboard:latest"
        PATH = "C:\\Users\\nares\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;${env.PATH}"
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/dhanushs2024c-sys/college-placement-portal.git'
            }
        }

        stage('Check Docker') {
            steps {
                bat '''
                    where docker
                    docker --version
                    docker info
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build --pull -t %DOCKER_IMAGE% .
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    bat '''
                        docker login -u "%USER%" -p "%PASS%"
                        docker push %DOCKER_IMAGE%
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    bat '''
                        set KUBECONFIG=%KUBECONFIG%
                        kubectl apply -f deployment.yaml --validate=false
                        kubectl rollout restart deployment/placement-portal
                        kubectl rollout status deployment/placement-portal --timeout=120s
                        kubectl get pods -l app=placement-portal
                    '''
                }
            }
        }
    }
}
