pipeline {
    agent any
    
    environment {
        // 1. اسم التطبيق (يجب أن يطابق اسم الـ Repository في Docker Hub)
        APP_NAME = 'web-app' 
        
        // 2. اسم المستخدم الخاص بك على Docker Hub
        DOCKER_HUB_USER = 'seif7atemmohamed' 
        
        // 3. المعرفات (IDs) التي قمت بإنشائها في Jenkins Credentials
        DOCKER_HUB_CREDS = 'docker-hub-creds'
        GIT_CREDS_ID = 'github-push-creds'
        
        REPO_URL = 'https://github.com/seif982/Mondaytask.git'
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def dockerTool = tool 'docker'
                    withEnv(["PATH+DOCKER=${dockerTool}/bin"]) {
                        // بناء الصورة محلياً برقم الـ Build الحالي
                        sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    def dockerTool = tool 'docker'
                    withEnv(["PATH+DOCKER=${dockerTool}/bin"]) {
                        withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS}", 
                                         passwordVariable: 'DOCKER_PASS', 
                                         usernameVariable: 'DOCKER_USER')]) {
                            
                            // تعديل سطر الـ login ليتوافق مع النسخة لديك
                            sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                            
                            // عمل Tag بالاسم الكامل لـ Docker Hub
                            sh "docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_HUB_USER}/${APP_NAME}:${BUILD_NUMBER}"
                            sh "docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_HUB_USER}/${APP_NAME}:latest"
                            
                            // رفع الصورة
                            sh "docker push ${DOCKER_HUB_USER}/${APP_NAME}:${BUILD_NUMBER}"
                            sh "docker push ${DOCKER_HUB_USER}/${APP_NAME}:latest"
                        }
                    }
                }
            }
        }

        stage('Run Locally') {
            steps {
                script {
                    def dockerTool = tool 'docker'
                    withEnv(["PATH+DOCKER=${dockerTool}/bin"]) {
                        sh """
                            # مسح أي كونتينر قديم بنفس الاسم
                            docker rm -f ${APP_NAME}-container || true
                            # تشغيل التطبيق على بورت 5000
                            docker run -d -p 5000:5000 --name ${APP_NAME}-container ${APP_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Push Tag to GitHub') {
            steps {
                // التأكد من وجود Credentials لـ GitHub باسم github-push-creds
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDS_ID}", 
                                 passwordVariable: 'GIT_TOKEN', 
                                 usernameVariable: 'GIT_USER')]) {
                    script {
                        sh """
                            git config user.email "seif@example.com"
                            git config user.name "seif7atemmohamed"
                            
                            # تحديث الـ Remote URL لإضافة الـ Token لعملية الـ Push
                            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/seif982/Mondaytask.git
                            
                            # إنشاء Tag للنسخة
                            git tag -a "v${BUILD_NUMBER}" -m "Stable Build ${BUILD_NUMBER}"
                            
                            # الرفع لـ GitHub
                            git push origin --tags
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Congratulations! Build, Push to DockerHub, Run, and GitHub Tagging are all DONE."
        }
        failure {
            echo "Something went wrong. Please check the Console Output."
        }
    }
}
