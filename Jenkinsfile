pipeline {
    agent any
    environment {
        DOCKERHUB_USER = 'ebrahimrhh'
        DOCKERHUB_PASS = credentials('dockerhub') // ده الـ ID اللي انت مخزنه في Jenkins
    }
    stages {
        stage('Build Backend Image') {
            steps { 
                dir('Backend') { 
                    sh 'docker build -t $DOCKERHUB_USER/phishing-backend:latest .' 
                } 
            }
        }
        stage('Build Frontend Image') {
            steps { 
                dir('my-nginx') { 
                    sh 'docker build -t $DOCKERHUB_USER/phishing-frontend:latest .' 
                } 
            }
        }
        stage('Push Images') {
            steps {
                sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                sh 'docker push $DOCKERHUB_USER/phishing-backend:latest'
                sh 'docker push $DOCKERHUB_USER/phishing-frontend:latest'
            }
        }
        stage('Clean Local Images') {
            steps {
                sh 'docker rmi $DOCKERHUB_USER/phishing-backend:latest || true'
                sh 'docker rmi $DOCKERHUB_USER/phishing-frontend:latest || true'
            }
        }
        stage('Update Deployment YAMLs') {
            steps {
                sh '''
                sed -i "s|image: .*phishing-backend.*|image: $DOCKERHUB_USER/phishing-backend:latest|" project-files/backend-deployment.yaml
                sed -i "s|image: .*phishing-frontend.*|image: $DOCKERHUB_USER/phishing-frontend:latest|" project-files/frontend-deployment.yaml
                '''
            }
        }
        stage('Push Updates to GitHub') {
            steps {
                dir('project-files') {
                    sh '''
                    git add .
                    git commit -m "Update images for ArgoCD deployment"
                    git push origin main
                    '''
                }
            }
        }
    }
}
