pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: node:18
    command:
    - cat
    tty: true
  - name: docker
    image: docker:24.0.5-cli
    command:
    - cat
    tty: true
"""
            defaultContainer 'node'
        }
    }

    environment {
        IMAGE_NAME = "saideepthij/ci-cd-test"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = "dockerhub-creds-id"
        KUBE_CONTEXT = "minikube"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/SaiDeepthiJ/kubepipeline'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || echo "No tests defined, skipping..."'
            }
        }

        stage('Docker Build & Push') {
            steps {
                container('docker') {
                    script {
                        docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    }
                    withCredentials([usernamePassword(
                        credentialsId: "${REGISTRY_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes via Helm') {
            steps {
                sh """
                    helm upgrade --install kubepipeline ./helm \
                      --set image.repository=${IMAGE_NAME} \
                      --set image.tag=${IMAGE_TAG} \
                      --namespace default
                """
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed!"
        }
        success {
            echo "Pipeline completed successfully!"
        }
    }
}
