pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/sandeshchalise"
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
                withCredentials([usernamePassword(
                    credentialsId: 'fd478bba-2d0e-4e5e-a268-af7839898698',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                sh '''
                docker build -t $REGISTRY/frontend:${BUILD_NUMBER} ./app-repo/frontend
                docker push $REGISTRY/frontend:${BUILD_NUMBER}
                '''
            }
        }

        stage('Build & Push Backend') {
            steps {
                sh '''
                docker build -t $REGISTRY/backend:${BUILD_NUMBER} ./app-repo/backend
                docker push $REGISTRY/backend:${BUILD_NUMBER}
                '''
            }
        }

        stage('Build & Push MySQL') {
            steps {
                sh '''
                docker build -t $REGISTRY/mysql:${BUILD_NUMBER} ./app-repo/mysql
                docker push $REGISTRY/mysql:${BUILD_NUMBER}
                '''
            }
        }

        stage('Update GitOps Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '0837ae32-3a05-4dbb-856e-c6add3bb67c7',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {

                    sh '''
                    rm -rf deploy-repo
                    git clone https://$GIT_USER:$GIT_PASS@github.com/sandeshchalise/multitier-k3s-gitops-repo.git deploy-repo
                    cd deploy-repo

                    # Update images with build number (versioned deployments)
                    sed -i "s|image:.*frontend.*|image: $REGISTRY/frontend:${BUILD_NUMBER}|g" frontend-deployment.yaml
                    sed -i "s|image:.*backend.*|image: $REGISTRY/backend:${BUILD_NUMBER}|g" backend-deployment.yaml
                    sed -i "s|image:.*mysql.*|image: $REGISTRY/mysql:${BUILD_NUMBER}|g" mysql-deployment.yaml

                    git config user.email "jenkins@ci.local"
                    git config user.name "Jenkins CI"

                    git add -A

                    if git diff --cached --quiet; then
                        echo "No changes to commit"
                    else
                        git commit -m "Update images to build ${BUILD_NUMBER}"
                        git push
                    fi
                    '''
                }
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
