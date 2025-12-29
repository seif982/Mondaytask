pipeline {
    agent any
    environment {
        APP_NAME = 'web-app'
        REPO_URL = 'https://github.com/seif982/Mondaytask.git'
    }

    stages {
        stage('Getting Repo files') {
            steps {
                // تأكد أن credentialsId 'github' موجودة فعلاً في Jenkins
                git branch: "main", url: "${REPO_URL}"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // استدعاء أداة Docker اللي عرفناها في الـ Tools
                    def dockerTool = tool 'docker'
                    withEnv(["PATH+DOCKER=${dockerTool}/bin"]) {
                        sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
                    }
                }
            }
        }
        
        stage('Run Docker Container') {
            steps {
                script {
                    def dockerTool = tool 'docker'
                    withEnv(["PATH+DOCKER=${dockerTool}/bin"]) {
                        // استخدام double quotes "" للسماح لـ Groovy بالتعامل مع المتغيرات
                        sh """
                            docker run -p 5000:5000 \
                            --name ${APP_NAME}-${BUILD_NUMBER} \
                            -d ${APP_NAME}:${BUILD_NUMBER}
                            docker ps
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
