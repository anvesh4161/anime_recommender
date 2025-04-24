pipeline {
    agent any

    environment {
        VENV_DIR = 'anime'
        GCP_PROJECT = 'plasma-datum-454301-t9'
        GCLOUD_PATH = "/var/jenkins_home/google-cloud-sdk/bin"
        KUBECTL_AUTH_PLUGIN = "/usr/lib/google-cloud-sdk/bin"
    }

    stages {
        stage("Cloning from GitHub...") {
            steps {
                echo "Cloning from GitHub..."
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/anvesh4161/anime_recommender.git'
            }
        }

        stage("Making a virtual environment...") {
            steps {
                script {
                    echo "Setting up virtual environment..."
                    sh '''
                        set -e
                        python -m venv ${VENV_DIR}
                        . ${VENV_DIR}/bin/activate
                        pip install --upgrade pip
                        pip install -e .
                        pip install dvc
                    '''
                }
            }
        }

        stage("DVC Pull") {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    echo "Pulling data with DVC..."
                    sh '''
                        set -e
                        . ${VENV_DIR}/bin/activate
                        dvc pull
                    '''
                }
            }
        }

        stage("Build and push image to GCR") {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    echo "Building and pushing Docker image to GCR..."
                    sh '''
                        set -e
                        export PATH=$PATH:${GCLOUD_PATH}
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${GCP_PROJECT}
                        gcloud auth configure-docker --quiet
                        docker build -t gcr.io/${GCP_PROJECT}/anime_recommender:latest .
                        docker push gcr.io/${GCP_PROJECT}/anime_recommender:latest
                    '''
                }
            }
        }

        stage("Deploying to Kubernetes...") {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    echo "Deploying to Kubernetes cluster..."
                    sh '''
                        set -e
                        export PATH=$PATH:${GCLOUD_PATH}:${KUBECTL_AUTH_PLUGIN}
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${GCP_PROJECT}
                        gcloud container clusters get-credentials ml-app-cluster --region us-central1
                        kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
