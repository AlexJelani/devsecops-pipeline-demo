pipeline {
    agent any
    environment {
        REGISTRY = '131.186.56.105:8081/repository/docker-hosted'
        IMAGE_NAME = "${REGISTRY}/alexjelani/devsecops-demo"
        NEXUS_CREDENTIALS = credentials('nexus-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'http://131.186.56.105:3000/alexjelani/devsecops-demo.git', branch: 'main'
            }
        }
        stage('Unit Testing') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        stage('Build Application') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: false
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:$GIT_COMMIT .'
                sh 'docker login -u $NEXUS_CREDENTIALS_USR -p $NEXUS_CREDENTIALS_PSW ${REGISTRY}'
                sh 'docker push ${IMAGE_NAME}:$GIT_COMMIT'
            }
        }
        stage('Update Manifest') {
            steps {
                sh '''
                sed -i "s|image: .*|image: ${IMAGE_NAME}:$GIT_COMMIT|" k8s/deployment.yaml
                git config user.name "jenkins"
                git config user.email "jenkins@lab"
                git add k8s/deployment.yaml
                git commit -m "Update image tag to $GIT_COMMIT"
                git push http://$NEXUS_CREDENTIALS_USR:$NEXUS_CREDENTIALS_PSW@131.186.56.105:3000/alexjelani/devsecops-demo.git
                '''
            }
        }
    }
}