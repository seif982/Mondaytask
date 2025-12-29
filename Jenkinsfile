pipeline {
    agent any
    environment {
        APP_NAME = 'web-app'
        REPO_URL = 'https://github.com/seif982/Mondaytask.git'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // سطر استدعاء الأداة - لازم الاسم يطابق اللي كتبته في الـ Tools
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
                        sh """
                            # مسح أي كونتينر قديم بنفس الاسم عشان ما يحصلش تعارض ports
                            docker rm -f ${APP_NAME}-container || true
                            
                            docker run -p 5000:5000 \
                            --name ${APP_NAME}-container \
                            -d ${APP_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed!' }
    }
}
