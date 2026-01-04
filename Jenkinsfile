pipeline {
    agent any

    environment {
        PROJECT_ID = "project-d64746f5-a7db-400d-898"
        REGION = "asia-south1"
        SERVICE_NAME = "todo-app"
        REPO_NAME = "todo-app-registry"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${SERVICE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                echo "Compiling application..."
                sh 'mvn clean compile'
            }
        }

        stage('Unit Tests') {
            steps {
                echo "Running unit tests..."
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo "Packaging Spring Boot JAR..."
                sh 'mvn package -DskipTests'
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GCLOUD_KEY')]) {
                    sh '''
                      gcloud auth activate-service-account --key-file=$GCLOUD_KEY
                      gcloud config set project ${PROJECT_ID}
                      gcloud auth configure-docker ${REGION}-docker.pkg.dev -q
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t ${IMAGE_URI} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing image to Artifact Registry..."
                sh 'docker push ${IMAGE_URI}'
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                echo "Deploying to Cloud Run..."
                sh '''
                  gcloud run deploy ${SERVICE_NAME} \
                    --image ${IMAGE_URI} \
                    --region ${REGION} \
                    --platform managed \
                    --allow-unauthenticated \
                    --port 8080
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
