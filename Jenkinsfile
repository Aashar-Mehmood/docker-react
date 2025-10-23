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
                powershell '''
                    Write-Host "Creating Dockerrun.aws.json..."

                    $json = @{
                        AWSEBDockerrunVersion = 2
                        containerDefinitions = @(
                            @{
                                name = "reactapp"
                                image = "$env:AWS_ACCOUNT_ID.dkr.ecr.$env:AWS_REGION.amazonaws.com/$env:ECR_REPO:latest"
                                essential = $true
                                memory = 256
                                portMappings = @(@{ containerPort = 80 })
                            }
                        )
                    } | ConvertTo-Json -Depth 5

                    $json | Out-File -FilePath Dockerrun.aws.json -Encoding utf8

                    Write-Host "Packaging Dockerrun.aws.json for deployment..."
                    Compress-Archive -Path Dockerrun.aws.json -DestinationPath deploy.zip -Force

                    Write-Host "Using EB environment..."
                    eb use $env:EB_ENV_NAME

                    Write-Host "Deploying to Elastic Beanstalk..."
                    eb deploy
                '''
            }
        }

    }
}
