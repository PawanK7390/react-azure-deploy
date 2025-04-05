pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'jenkins-sp'
        RESOURCE_GROUP = 'rg-react'
        APP_SERVICE_NAME = 'reactwebappjenkins838796'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/PawanK7390/react-azure-deploy.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    bat 'terraform init || exit /b'
                }
            }
        }

        stage('Terraform Import Existing Resources') {
            steps {
                dir('terraform') {
                    // Tries to import the RG if it already exists; continues if it doesn’t
                    bat '''
                        terraform state show azurerm_resource_group.rg >nul 2>&1 || ^
                        terraform import azurerm_resource_group.rg /subscriptions/eea7dd66-806c-47a7-912f-2e3f1af71f5e/resourceGroups/rg-react
                    '''
                }
            }
        }

        stage('Terraform Plan & Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform plan -out=tfplan || exit /b'
                    bat 'terraform apply -auto-approve tfplan || exit /b'
                }
            }
        }

        stage('Install Dependencies & Build') {
            steps {
                bat 'npm install || exit /b'
                bat 'npm run build || exit /b'
            }
        }

        stage('Package React Build') {
            steps {
                bat 'powershell -Command "if (Test-Path publish) { Remove-Item -Recurse -Force publish }"'
                bat 'powershell -Command "New-Item -ItemType Directory -Path publish"'
                bat 'powershell -Command "Copy-Item -Path build\\* -Destination publish -Recurse -Force"'
                bat 'powershell -Command "Compress-Archive -Path publish\\* -DestinationPath publish.zip -Force"'
            }
        }


        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat 'az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%'
                    bat 'az webapp deployment source config-zip --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src publish.zip'
                }
            }
        }
    }

    post {
        success {
            echo ' React App Deployed Successfully!'
        }
        failure {
            echo ' Deployment Failed. Check the logs above carefully.'
        }
    }
}
