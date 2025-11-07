pipeline {
    agent any
    stages {
        stage('Build Backend Image') {
            steps { dir('Backend') { sh 'docker build -t ebrahimrhh/phishing-backend:latest .' } }
        }
        stage('Build Frontend Image') {
            steps { dir('my-nginx') { sh 'docker build -t ebrahimrhh/phishing-frontend:latest .' } }
        }
        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push $DOCKERHUB_USER/phishing-backend:latest
                        docker push $DOCKERHUB_USER/phishing-frontend:latest
                    '''
                }
            }
        }
        stage('Clean Local Images') {
            steps {
                sh 'docker rmi ebrahimrhh/phishing-backend:latest || true'
                sh 'docker rmi ebrahimrhh/phishing-frontend:latest || true'
            }
        }
        stage('Update Deployment YAMLs') {
            steps {
                sh '''
                sed -i "s|image: .*phishing-backend.*|image: ebrahimrhh/phishing-backend:latest|" project-files/backend-deployment.yaml
                sed -i "s|image: .*phishing-frontend.*|image: ebrahimrhh/phishing-frontend:latest|" project-files/frontend-deployment.yaml
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
