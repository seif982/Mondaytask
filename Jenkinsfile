pipeline {
    agent any
    
    environment {
        APP_NAME = 'web-app'
        REPO_URL = 'https://github.com/seif982/Mondaytask.git'
        // تأكد من إنشاء هذه الـ Credentials في جنكينز كما شرحنا سابقاً
        GIT_CREDS_ID = 'github-push-creds' 
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // استدعاء أداة Docker المعرفة في الـ Global Tool Configuration
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
                            # مسح أي كونتينر قديم بنفس الاسم لتجنب الخطأ
                            docker rm -f ${APP_NAME}-container || true
                            
                            # تشغيل الكونتينر الجديد على بورت 5000
                            docker run -d -p 5000:5000 --name ${APP_NAME}-container ${APP_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Push Tag to GitHub') {
            steps {
                // استخدام الـ Token اللي عملناه في الخطوة السابقة
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDS_ID}", 
                                 passwordVariable: 'GIT_TOKEN', 
                                 usernameVariable: 'GIT_USER')]) {
                    script {
                        sh """
                            # إعدادات الـ Git (غير الإيميل والاسم لبياناتك)
                            git config user.email "seif@example.com"
                            git config user.name "seif982"
                            
                            # تحديث الـ URL ليشمل الـ Token للـ Authentication
                            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/seif982/Mondaytask.git
                            
                            # إنشاء Tag برقم الـ Build
                            git tag -a "v${BUILD_NUMBER}" -m "Stable Build ${BUILD_NUMBER} by Jenkins"
                            
                            # رفع الـ Tag لـ GitHub
                            git push origin --tags
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully built, deployed, and tagged build #${BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline failed. Check Console Output for details."
        }
    }
}
