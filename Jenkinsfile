pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'jenkins-sp'
        RESOURCE_GROUP = 'rg-react'
        APP_SERVICE_NAME = 'reactwebappjenkins838796'
        ZIP_FILE = 'build.zip' 
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

        stage('Verify ZIP Exists') {
            steps {
                script {
                    def zipExists = fileExists("${env.ZIP_FILE}")
                    if (!zipExists) {
                        error("${env.ZIP_FILE} not found! Check if zipping succeeded.")
                    }
                }
            }
        }

        stage('Deploy to Azure via Kudu') {
            steps {
                withCredentials([
                    string(credentialsId: 'kudu-deploy-username', variable: 'KUDU_USER'),
                    string(credentialsId: 'kudu-deploy-password', variable: 'KUDU_PASS')
                ]) {
                    script {
                        def kuduUrl = "https://${env.APP_SERVICE_NAME}.scm.azurewebsites.net/api/zipdeploy"
                        def deployCmd = """
                        curl --fail -X POST "${kuduUrl}" ^
                            -u "${KUDU_USER}:${KUDU_PASS}" ^
                            --data-binary "@${env.ZIP_FILE}" ^
                            -H "Content-Type: application/zip"
                        """

                        bat deployCmd
                    }
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
