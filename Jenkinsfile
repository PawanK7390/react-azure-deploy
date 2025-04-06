pipeline {
    agent any

    environment {
        RESOURCE_GROUP = 'rg-react'
        APP_SERVICE_NAME = 'reactwebappjenkins838796'
        ZIP_FILE = 'build.zip'
        KUDU_USER = 'reactwebappjenkins838796'
        KUDU_PASS = '96x1BuPphQAmwxyjrArAgqxw2HGndDaemgjTRPKpZkl5znjy97JltAjcJZZq'
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

        stage('Terraform Import') {
            steps {
                dir('terraform') {
                    bat '''
                        terraform import azurerm_resource_group.rg "/subscriptions/eea7dd66-806c-47a7-912f-2e3f1af71f5e/resourceGroups/rg-react" || exit /b
                        terraform import azurerm_service_plan.asp "/subscriptions/eea7dd66-806c-47a7-912f-2e3f1af71f5e/resourceGroups/rg-react/providers/Microsoft.Web/serverFarms/react-app-plan" || exit /b
                        terraform import azurerm_linux_web_app.react_app "/subscriptions/eea7dd66-806c-47a7-912f-2e3f1af71f5e/resourceGroups/rg-react/providers/Microsoft.Web/sites/reactwebappjenkins838796" || exit /b
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

        stage('Install Dependencies & Build React') {
            steps {
                bat 'npm install || exit /b'
                bat 'npm run build || exit /b'
            }
        }

        stage('Check Build Folder') {
            steps {
                script {
                    def buildExists = fileExists('build\\index.html')
                    if (!buildExists) {
                        error("Build folder missing or empty. Make sure 'npm run build' succeeded.")
                    }
                }
            }
        }

        stage('Debug ZIP Path') {
            steps {
                echo "ZIP file will be created at: ${env.ZIP_FILE}"
            }
        }

        stage('Zip Build Folder') {
            steps {
                bat """
                    powershell -Command "if (Test-Path '${env.ZIP_FILE}') { Remove-Item -Force '${env.ZIP_FILE}' }"
                    powershell -Command "Compress-Archive -Path build\\* -DestinationPath '${env.ZIP_FILE}' -Force"
                """
            }
        }

        stage('Verify ZIP') {
            steps {
                script {
                    def zipExists = fileExists("${env.ZIP_FILE}")
                    if (!zipExists) {
                        error("${env.ZIP_FILE} not found! Check if zipping succeeded.")
                    }
                }
            }
        }

        stage('Deploy to Azure') {
            steps {
                script {
                    def kuduUrl = "https://${env.APP_SERVICE_NAME}.scm.azurewebsites.net/api/zipdeploy"
                    bat """
                        curl --fail -X POST "${kuduUrl}" ^
                            -u "${env.KUDU_USER}:${env.KUDU_PASS}" ^
                            --data-binary "@${env.ZIP_FILE}" ^
                            -H "Content-Type: application/zip"
                    """
                }
            }
        }
    }

    post {
        success {
            echo ' React App Deployed Successfully via Kudu ZIP Deploy!'
        }
        failure {
            echo ' Deployment Failed. Check the logs above carefully.'
        }
        always {
            cleanWs()
        }
    }
}
