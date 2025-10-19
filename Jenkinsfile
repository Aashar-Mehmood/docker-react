pipeline{
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Testing...'
                sh '''
                    docker build -t docker-react-test -f Dockerfile.dev . 
                    docker run docker-react-test npm run test -- --coverage
                '''
            }
        }
         stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}