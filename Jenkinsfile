pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        DOCKER_REGISTRY = "crpi-lounudtd1uqkxxn7.cn-shanghai.personal.cr.aliyuncs.com/my-app-space-2026"
        IMAGE_NAME = "my-java-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('Clean Workspace') {
            steps { cleanWs() }
        }
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'git@github.com:JackyGao2025/my-java-app.git',
                    credentialsId: 'github-ssh-key'
                echo '✅ 代码拉取完成'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
                echo '✅ Maven 构建完成'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                echo "✅ Docker 镜像构建完成"
            }
        }
        stage('Push Image to ACR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aliyun-acr-credentials',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh """
                        echo \$ACR_PASS | docker login --username=\$ACR_USER ${DOCKER_REGISTRY%%/*} --password-stdin
                        docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
                echo '✅ 镜像已推送到阿里云 ACR'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    sed 's|\${IMAGE_TAG}|${IMAGE_TAG}|g' deployment.yaml > deployment-resolved.yaml
                    kubectl apply -f deployment-resolved.yaml
                    kubectl apply -f service.yaml
                    kubectl rollout status deployment/my-java-app --timeout=180s
                """
                echo '✅ 部署完成'
            }
        }
        stage('Verify') {
            steps {
                sh """
                    sleep 10
                    kubectl get pods -l app=my-java-app
                    curl -s http://\$(minikube ip):30080
                """
                echo '✅ 验证完成'
            }
        }
    }
    post {
        success { echo '🎉 流水线执行成功！' }
        failure { echo '❌ 流水线执行失败，请查看日志' }
    }
}

