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
                    docker compose build
                '''
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aashar-aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                     sh '''
                        aws --version
                        aws ec2 describe-instances --region eu-north-1
                    '''
                }
               
            }
        }
        // stage('Run Locally') {
        //     steps {
        //         echo 'Starting Container Locally ...'
        //         sh '''
        //             docker compose up -d
        //         '''
        //     }
        // }
    }
}