pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        IMAGE_NAME = "my-java-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "my-java-container"
    }
    stages {
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
        stage('Deploy') {
            steps {
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
                sh "docker run -d --name ${CONTAINER_NAME} -p 8082:8080 ${IMAGE_NAME}:${IMAGE_TAG}"
                echo "✅ 容器部署完成"
            }
        }
         stage('Verify') {
            steps {
                sh """
                sleep 15
        # 检查容器是否在运行
               RUNNING=\$(docker inspect -f '{{.State.Running}}' ${CONTAINER_NAME})
               if [ "\$RUNNING" != "true" ]; then
               echo "Container is not running"
               exit 1
               fi
        # 检查端口映射
              docker port ${CONTAINER_NAME}
              echo "✅ 容器运行正常，端口映射正确"
              """
             }
         }
    }
}
