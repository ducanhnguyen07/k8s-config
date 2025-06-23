pipeline {
    agent {
        label 'kubeagent'
    }

    parameters {
        string(name: 'ref', defaultValue: '', description: 'Git ref from webhook payload')
    }

    environment {
        IMAGE_NAME = "anhnd2301/ecommerce-backend"
        GIT_TAG = "${params.ref.replace('refs/tags/', '')}"
    }

    stages {
        stage('Agent & Build Information') {
            steps {
                sh 'whoami'
                sh 'pwd'
                sh "echo 'Received Git ref: ${params.ref}'"
                sh "echo 'Parsed Git Tag: ${env.GIT_TAG}'"
            }
        }

        stage('Build & Push Docker Image by Tag') {
            when {
                expression {
                    return params.ref != null && params.ref.startsWith('refs/tags/')
                }
            }
            
            steps {
                echo "Checking out the correct tag: ${params.ref}"
                // Lệnh này sẽ ghi đè workspace hiện tại bằng code từ đúng tag
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${params.ref}"]],
                    userRemoteConfigs: [[
                        url: 'http://git.anhnd.vn/ecommerce/ecommerce.git',
                        credentialsId: 'jenkins-gitlab-user-account'
                    ]]
                ])

                container('docker') {
                    sh '''
                        echo "Preparing Docker credentials..."
                        mkdir -p /root/.docker
                        cp /tmp/docker-config-secret/.dockerconfigjson /root/.docker/config.json
                        echo "Credentials ready."
                    '''
                    
                    // Sử dụng biến ${env.GIT_TAG} đã được xử lý
                    echo "Building image for tag: ${env.GIT_TAG}"
                    dir('02-backend_spring-boot-rest-api') {
                        sh "docker build -t ${IMAGE_NAME}:${env.GIT_TAG} ."
                    }
                    
                    echo "Pushing image to Docker Hub: ${IMAGE_NAME}:${env.GIT_TAG}"
                    sh "docker push ${IMAGE_NAME}:${env.GIT_TAG}"
                    
                    echo "Build and Push for tag '${env.GIT_TAG}' completed successfully!"
                }
            }
        }
    }
}
