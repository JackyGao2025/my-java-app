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
        IMAGE_NAME = "my-java-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                // 注意：此处不再重复 checkout，后续 Checkout 阶段会拉取代码
            }
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
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                echo "✅ Docker 镜像构建完成: ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
        stage('Load Image to Minikube') {
            steps {
                sh "minikube image load ${IMAGE_NAME}:${IMAGE_TAG}"
                echo "✅ 镜像已加载到 minikube"
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    sed 's/\${IMAGE_TAG}/${IMAGE_TAG}/g' deployment.yaml > deployment-resolved.yaml
                    kubectl apply -f deployment-resolved.yaml
                    kubectl apply -f service.yaml
                """
                echo '✅ 部署清单已应用（Pod 将在后台更新）'
            }
        }
        stage('Verify') {
            steps {
                sh """
                    echo "等待 Pod 更新..."
                    sleep 30
                    kubectl get pods -l app=my-java-app
                """
                echo '✅ 部署完成，请手动检查 Pod 状态'
            }
        }
    }
    post {
        success { echo '🎉 整个 CI/CD 流水线执行成功！' }
        failure { echo '❌ 流水线执行失败，请查看日志' }
    }
}
