pipeline {
    agent any
    options {
        skipDefaultCheckout(true)   // 禁用默认的 checkout
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
                cleanWs()   // 清空工作区
                checkout scm   // 重新从 SCM 拉取
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
        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    # 替换 deployment.yaml 中的镜像标签
                    sed 's/\${IMAGE_TAG}/${IMAGE_TAG}/g' deployment.yaml > deployment-resolved.yaml
                    # 应用部署清单
                    kubectl apply -f deployment-resolved.yaml
                    kubectl apply -f service.yaml
                    # 等待滚动更新完成
                    kubectl rollout status deployment/my-java-app
                """
                echo '✅ 应用已部署到 Kubernetes'
            }
        }
        stage('Verify') {
            steps {
                sh """
                    NODE_PORT=\$(kubectl get svc my-java-app-service -o jsonpath='{.spec.ports[0].nodePort}')
                    for i in \$(seq 1 30); do
                        if curl -s http://localhost:\${NODE_PORT} > /dev/null; then
                            echo "✅ 服务启动成功"
                            break
                        fi
                        echo "⏳ 等待服务启动... (\${i}/30)"
                        sleep 2
                    done
                    curl -s http://localhost:\${NODE_PORT}
                """
            }
        }
    }
    post {
        success { echo '🎉 整个 CI/CD 流水线执行成功！' }
        failure { echo '❌ 流水线执行失败，请查看日志' }
    }
}
