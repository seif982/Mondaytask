pipeline {
    agent any
    
    environment {
        // 1. اسم التطبيق (لازم يطابق اسم الـ Repo في Docker Hub)
        APP_NAME = 'web-app' 
        
        // 2. اسم المستخدم بتاعك على Docker Hub
        DOCKER_HUB_USER = 'seif982' 
        
        // 3. المعرفات (IDs) اللي إنت عملتها في الـ Credentials بتاعة جنكينز
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
                        // بناء الصورة محلياً
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
                            
                            // تسجيل الدخول والرفع
                            sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                            sh "docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_HUB_USER}/${APP_NAME}:${BUILD_NUMBER}"
                            sh "docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_HUB_USER}/${APP_NAME}:latest"
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
                            docker rm -f ${APP_NAME}-container || true
                            docker run -d -p 5000:5000 --name ${APP_NAME}-container ${APP_NAME}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Push Tag to GitHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDS_ID}", 
                                 passwordVariable: 'GIT_TOKEN', 
                                 usernameVariable: 'GIT_USER')]) {
                    script {
                        sh """
                            git config user.email "seif7atem900@gmail.com"
                            git config user.name "seif982"
                            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/seif982/Mondaytask.git
                            git tag -a "v${BUILD_NUMBER}" -m "Stable Build ${BUILD_NUMBER}"
                            git push origin --tags
                        """
                    }
                }
            }
        }
    }
}
