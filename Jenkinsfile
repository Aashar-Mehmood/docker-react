pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        ECR_REPO = 'docker-react-frontend'
        EB_APP_NAME = 'docker-react-frontend'
        EB_ENV_NAME = 'docker-react-frontend-env'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t %ECR_REPO%:latest .
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                bat '''
                    for /f "delims=" %%i in ('aws sts get-caller-identity --query Account --output text') do set AWS_ACCOUNT_ID=%%i
                    set ECR_URL=%AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%
                    
                    aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com
                    docker tag %ECR_REPO%:latest %ECR_URL%:latest
                    docker push %ECR_URL%:latest
                '''
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                bat '''
                    echo Creating Dockerrun.aws.json...

                    rem === Create Dockerrun.aws.json using forward slashes only ===
                    (
                        echo {
                        echo   "AWSEBDockerrunVersion": 2,
                        echo   "containerDefinitions": [
                        echo     {
                        echo       "name": "reactapp",
                        echo       "image": "%AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:latest",
                        echo       "essential": true,
                        echo       "memory": 256,
                        echo       "portMappings": [ { "containerPort": 80 } ]
                        echo     }
                        echo   ]
                        echo }
                    ) > Dockerrun.aws.json

                    rem === Ensure Unix-style slashes for EB CLI ===
                    powershell -Command "(Get-Content Dockerrun.aws.json) | ForEach-Object {$_ -replace '\\\\','/'} | Set-Content -Encoding UTF8 Dockerrun.aws.json"

                    rem === Initialize Elastic Beanstalk if not already done ===
                    if not exist ".elasticbeanstalk" (
                        eb init %EB_APP_NAME% --region %AWS_REGION% --platform "Docker"
                    )

                    rem === Set environment ===
                    eb use %EB_ENV_NAME%

                    rem === Deploy to Elastic Beanstalk ===
                    eb deploy --staged
                '''
            }
        }
    }
}
