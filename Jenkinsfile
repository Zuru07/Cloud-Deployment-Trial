pipeline {
    agent any
    environment {
        GCP_PROJECT   = 'eastern-rider-477015-v1'   
        IMAGE_NAME    = 'hello-gcp'
        IMAGE_TAG     = "${env.BUILD_NUMBER}"
        REGISTRY_URL  = "${GCP_PROJECT}-docker.pkg.dev/${GCP_PROJECT}/my-repo/${IMAGE_NAME}"
    }
    stages {
        stage('Checkout') {
            steps { git url: 'https://github.com/Zuru07/CIA2-AWS-Project.git', branch: 'main' }
        }
        stage('Build & Test') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t ${REGISTRY_URL}:${IMAGE_TAG} ."
                sh "docker tag ${REGISTRY_URL}:${IMAGE_TAG} ${REGISTRY_URL}:latest"
            }
        }
        stage('Push to Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcp-jenkins-sa', variable: 'GC_KEY')]) {
                    sh "gcloud auth activate-service-account --key-file=${GC_KEY}"
                    sh "gcloud auth configure-docker ${GCP_PROJECT}-docker.pkg.dev"
                    sh "docker push ${REGISTRY_URL}:${IMAGE_TAG}"
                    sh "docker push ${REGISTRY_URL}:latest"
                }
            }
        }
        stage('Deploy to Cloud Run') {
            steps {
                withCredentials([file(credentialsId: 'gcp-jenkins-sa', variable: 'GC_KEY')]) {
                    sh "gcloud auth activate-service-account --key-file=${GC_KEY}"
                    sh """
                    gcloud run deploy hello-gcp \
                      --image=${REGISTRY_URL}:latest \
                      --platform=managed \
                      --region=us-central1 \
                      --allow-unauthenticated \
                      --project=${GCP_PROJECT} \
                      --quiet
                    """
                }
            }
        }
    }
    post {
        success { echo 'Pipeline succeeded!' }
        failure { echo 'Pipeline failed!' }
    }
}