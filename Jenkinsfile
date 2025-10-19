pipeline{
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Testing...'
                sh '''
                    docker build -t docker-react-test-image -f Dockerfile.dev . 
                    docker run --name docker-react-test-container docker-react-test-image npm run test -- --coverage
                    docker stop docker-react-test-container || true
                    docker rm docker-react-test-container || true
                '''
            }
        }
         stage('Build') {
            steps {
                echo 'Building...'
                sh '''
                    docker compose up
                '''
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}