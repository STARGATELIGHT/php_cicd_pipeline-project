pipeline {
    agent any
    environment {
        IMAGE_NAME = "php-app"
        IMAGE_TAG = "latest"
        DOCKER_REGISTRY = "docker.io/your-username"  // or use Nexus Docker registry URL
        SONARQUBE_URL = "http://<SONARQUBE_EC2_PRIVATE_IP>:9000"
        NEXUS_RAW_URL = "http://<NEXUS_EC2_PRIVATE_IP>:8081/repository/raw-hosted/"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-username/php-ci-app.git'
            }
        }

        stage('Code Integration') {
            steps {
                sh 'php -l index.php'
            }
        }

        stage('Package Artifact') {
            steps {
                echo 'Packaging application...'
                sh 'tar -czf app.tar.gz .'
            }
        }

        stage('SonarQube Code Quality Scan') {
            steps {
                sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=php-app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONARQUBE_URL \
                        -Dsonar.login=your_sonar_token
                '''
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-creds',
                    passwordVariable: 'NEXUS_PASS',
                    usernameVariable: 'NEXUS_USER'
                )]) {
                    sh '''
                        curl -v --user $NEXUS_USER:$NEXUS_PASS \
                        --upload-file app.tar.gz \
                        $NEXUS_RAW_URL/app.tar.gz
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    passwordVariable: 'DOCKER_PASSWORD',
                    usernameVariable: 'DOCKER_USERNAME'
                )]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login $DOCKER_REGISTRY -u $DOCKER_USERNAME --password-stdin
                        docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                        docker push $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }
}

