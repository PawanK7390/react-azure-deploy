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
                        def result = bat(
                            script: 'terraform state show azurerm_resource_group.rg',
                            returnStatus: true
                        )
                        if (result != 0) {
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

        stage(' Build Folder') {
            steps {
                script {
                    def buildExists = fileExists('build/index.html')
                    if (!buildExists) {
                        error("Build folder missing or empty. Make sure 'npm run build' succeeded.")
                    }
                }
            }
        }

        stage('Zip Build Folder') {
            steps {
                bat 'powershell -Command "if (Test-Path publish.zip) { Remove-Item -Force publish.zip }"'
                bat 'powershell -Command "Compress-Archive -Path build\\* -DestinationPath publish.zip -Force"'
            }
        }

        stage('Deploy to Azure ') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat 'az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%'

                    // Optional: Ensure no server-side build occurs
                    bat 'az webapp config appsettings set --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --settings SCM_DO_BUILD_DURING_DEPLOYMENT=false'

                    // Deploy zip package (React build)
                    bat 'az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path publish.zip --type zip || exit /b'
                }
            }
        }
    }

    post {
        success {
            echo ' React App Deployed Successfully using config-zip!'
        }
        failure {
            echo ' Deployment Failed. Check the logs above carefully.'
        }
    }
}
