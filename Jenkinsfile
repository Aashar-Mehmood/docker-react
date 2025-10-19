pipeline{
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Testing...'
                docker build -t docker-react-test -f Dockerfile.dev . 
                docker run -e CI=true docker-react-test npm run test -- --coverage
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