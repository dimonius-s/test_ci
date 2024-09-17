pipeline {
    agent any
    environment {
        IMAGE = "nginx:latest"
        CONTAINER_NAME = "nginx_ci_test"
        HOST_PORT = "9889"
        CONTAINER_PORT = "80"
        INDEX_FILE = "index.html"
        REPO_URL = "https://github.com/dimonius-s/test_ci.git"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: "${REPO_URL}", branch: 'main'
            }
        }
        stage('Build and Run Nginx Container') {
            steps {
                sh "docker run --name \${CONTAINER_NAME} -d -p \${HOST_PORT}:\${CONTAINER_PORT} -v \$(pwd)/\${INDEX_FILE}:/usr/share/nginx/html/index.html \${IMAGE}"
            }
        }
        stage('Check HTTP Response Code') {
            steps {
                script {
                    def response = sh(script: "curl -o /dev/null -s -w \"%{http_code}\" http://localhost:${HOST_PORT}", returnStdout: true).trim()
                    if (response != '200') {
                        error("HTTP response code is not 200: ${response}")
                    }
                }
            }
        }
        stage('Verify MD5 Hash') {
            steps {
                script {
                    def local_md5 = sh(script: "md5sum ${INDEX_FILE} | cut -d ' ' -f 1", returnStdout: true).trim()
                    def remote_md5 = sh(script: "curl -s http://localhost:${HOST_PORT}/${INDEX_FILE} | md5sum | cut -d ' ' -f 1", returnStdout: true).trim()
                    if (local_md5 != remote_md5) {
                        error("MD5 checksums do not match. Local: ${local_md5}, Remote: ${remote_md5}")
                    }
                }
            }
        }
    }
    post {
        always {
            sh "docker rm -f ${CONTAINER_NAME}"
        }
        success {
            sh 'curl -X POST -H "Content-Type: application/json" -d \'{"chat_id": "549465611", "text": "[🔥SUCCESS] api build success 🔥🔥🔥🔥🔥🔥🔥🔥🔥🔥!", "disable_notification": false}\' "https://api.telegram.org/bot7290400383:AAFd1Alt9agErk7cdZY_WK8_kH8vmFKODfE/sendMessage"'
        }
        failure {
            sh 'curl -X POST -H "Content-Type: application/json" -d \'{"chat_id": "549465611", "text": "[💀FAILED] api build failed  😭😭😭😭😭😭😭😭😭😭😭😭!", "disable_notification": false}\' "https://api.telegram.org/bot7290400383:AAFd1Alt9agErk7cdZY_WK8_kH8vmFKODfE/sendMessage"'
        }
    }
}
