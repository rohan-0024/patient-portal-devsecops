pipeline {

    agent any

    environment {

        IMAGE = "rohannn004/patient-portal"

        KUBECONFIG = "/home/ec2-user/.kube/config"
    }

    stages {

        stage('Checkout') {

            steps {
                checkout scm
            }
        }

        stage('Secret Scan') {

            steps {

                sh '''
                    trivy fs \
                    --scanners secret \
                    --exit-code 1 .
                '''
            }
        }

        stage('Dependency / Config Scan') {

            steps {

                sh '''
                    trivy fs \
                    --scanners vuln,misconfig \
                    --severity HIGH,CRITICAL \
                    --exit-code 0 .
                '''
            }
        }

        stage('Build Image') {

            steps {

                sh '''
                    docker build \
                    -t $IMAGE:${BUILD_NUMBER} \
                    .
                '''
            }
        }

        stage('Image Vulnerability Scan') {

            steps {

                sh '''
                    trivy image \
                    --severity CRITICAL \
                    --exit-code 1 \
                    $IMAGE:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Image') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'U',
                        passwordVariable: 'P'
                    )
                ]) {

                    sh '''
                        echo $P | docker login \
                        -u $U \
                        --password-stdin
                    '''

                    sh '''
                        docker push \
                        $IMAGE:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy Securely') {

    steps {

        sh "sed -i 's#rohannn004/patient-portal:latest#${IMAGE}:${BUILD_NUMBER}#' k8s/secure-deployment.yaml"

        sh "kubectl apply -f k8s/secure-deployment.yaml"

        sh "kubectl rollout status deployment/patient-portal --timeout=120s"
    }
}
    }
}
