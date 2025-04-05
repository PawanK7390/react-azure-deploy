pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'jenkins-sp'
        RESOURCE_GROUP = 'rg-react'
        APP_SERVICE_NAME = 'reactwebappjenkins838'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/PawanK7390/react-azure-deploy.git'
            }
        }

        // ---------- TERRAFORM (OPTIONAL) ----------
        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    bat 'terraform init'
                }
            }
        }

        stage('Terraform Plan and Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform plan -out=tfplan'
                    bat 'terraform apply -auto-approve tfplan'
                }
            }
        }

        // ---------- BUILD REACT ----------
        stage('Install Dependencies & Build') {
            steps {
                bat 'npm install'
                bat 'npm run build'
            }
        }

        // ---------- PACKAGE APP ----------
        stage('Package Application') {
            steps {
                bat 'powershell -Command "Remove-Item -Recurse -Force publish -ErrorAction SilentlyContinue"'
                bat 'powershell -Command "Copy-Item -Recurse -Path build -Destination publish"'
                bat 'powershell -Command "Compress-Archive -Path publish\\* -DestinationPath publish.zip -Force"'
            }
        }

        // ---------- DEPLOY TO AZURE ----------
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
            echo ' Deployment Failed. Check logs above.'
        }
    }
}
