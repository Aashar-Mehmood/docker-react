pipeline{
    agent any

    environment {
        EB_APP_NAME = 'docker-react-frontend'
        EB_ENV_NAME = 'docker-react-frontend-env'
        AWS_REGION = 'eu-north-1'
    }

    stages {
        // stage('Test') {
        //     steps {
        //         echo 'Testing...'
        //         sh '''
        //             docker build -t docker-react-test-image -f Dockerfile.dev . 
        //             docker run --name docker-react-test-container docker-react-test-image npm run test -- --coverage
        //             docker stop docker-react-test-container || true
        //             docker rm docker-react-test-container || true
        //         '''
        //     }
        // }
        stage('Build') {
            steps {
                echo 'Building...'
                bat 'npm install'
                bat 'npm run build'
            }
        }
        stage('Prepare Artifact') {
            steps {
                sh '''
                    zip -r deploy.zip build/
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                                    credentialsId: 'aashar-aws-creds', 
                                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
                        S3_BUCKET="elasticbeanstalk-$AWS_REGION-$ACCOUNT_ID"
                        VERSION_LABEL="v${BUILD_NUMBER}"

                        # Upload ZIP to S3
                        aws s3 cp deploy.zip s3://$S3_BUCKET/$EB_APP_NAME-$VERSION_LABEL.zip

                        # Create new application version
                        aws elasticbeanstalk create-application-version \
                        --application-name $EB_APP_NAME \
                        --version-label $VERSION_LABEL \
                        --source-bundle S3Bucket=$S3_BUCKET,S3Key=$EB_APP_NAME-$VERSION_LABEL.zip

                        # Update environment
                        aws elasticbeanstalk update-environment \
                        --environment-name $EB_ENV_NAME \
                        --version-label $VERSION_LABEL
                    '''
                }
            }
        }
        
    }
}