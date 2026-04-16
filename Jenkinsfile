pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/sandeshchalise"   // your Docker Hub namespace
        APP_REPO = "https://github.com/sandeshchalise/multitier-k3s-app-repo.git"
        DEPLOY_REPO = "https://github.com/sandeshchalise/multitier-k3s-gitops-repo.git"
    }

    stages {
        stage('Clone Application Repo') {
            steps {
                sh 'rm -rf app-repo && git clone $APP_REPO app-repo'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                sh 'docker build -t $REGISTRY/frontend:latest ./app-repo/frontend'
                sh 'docker push $REGISTRY/frontend:latest'
            }
        }

        stage('Build & Push Backend') {
            steps {
                sh 'docker build -t $REGISTRY/backend:latest ./app-repo/backend'
                sh 'docker push $REGISTRY/backend:latest'
            }
        }

        stage('Build & Push MySQL') {
            steps {
                sh 'docker build -t $REGISTRY/mysql:latest ./app-repo/mysql'
                sh 'docker push $REGISTRY/mysql:latest'
            }
        }

        stage('Update GitOps Repo') {
            steps {
                sh '''
                rm -rf deploy-repo
                git clone $DEPLOY_REPO
                cd deploy-repo

                sed -i "s|image: .*/frontend:.*|image: $REGISTRY/frontend:latest|" frontend-deployment.yaml
                sed -i "s|image: .*/backend:.*|image: $REGISTRY/backend:latest|" backend-deployment.yaml
                sed -i "s|image: .*/mysql:.*|image: $REGISTRY/mysql:latest|" mysql-deployment.yaml

                git config user.email "jenkins@ci.local"
                git config user.name "Jenkins CI"
                git commit -am "Update images to latest build"
                git push
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

