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
                    script {
                        def stateExists = bat(
                            script: 'terraform state show azurerm_resource_group.rg >nul 2>&1',
                            returnStatus: true
                        ) == 0

                        if (!stateExists) {
                            bat 'terraform import azurerm_resource_group.rg /subscriptions/eea7dd66-806c-47a7-912f-2e3f1af71f5e/resourceGroups/rg-react || exit /b'
                        }
                    }
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

        stage('Install Dependencies & Build React') {
            steps {
                bat 'npm install || exit /b'
                bat 'npm run build || exit /b'
            }
        }

        stage('Zip Build Folder Only') {
            steps {
                bat 'powershell -Command "if (Test-Path publish.zip) { Remove-Item publish.zip -Force }"'
                bat 'powershell -Command "Compress-Archive -Path build\\* -DestinationPath publish.zip -Force"'
            }
        }

        stage('Deploy to Azure App Service') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat 'az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%'
                    bat 'az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path publish.zip --type zip'
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
