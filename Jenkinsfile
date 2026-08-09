pipeline {
    agent any

    environment {
        APP_NAME = 'fluidai-devops-challenge'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "fluidai-devops-challenge:${BUILD_NUMBER}"
        NAMESPACE = 'fluidai'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "=== Java ==="
                    java -version

                    echo "=== Maven ==="
                    mvn -version

                    echo "=== Docker ==="
                    docker --version

                    echo "=== kubectl ==="
                    kubectl version --client

                    echo "=== Kubernetes Context ==="
                    kubectl config current-context
                '''
            }
        }

        stage('Build & Test') {
            steps {
                dir('app') {
                    sh '''
                        echo "=== Maven Build & Tests ==="
                        mvn clean package
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e

                    echo "=== Building Docker Image in Minikube ==="

                    minikube image build \
                        -t ${IMAGE_NAME} \
                        ./app

                    echo "=== Verifying Docker Image ==="

                    eval $(minikube -p minikube docker-env)

                    docker image inspect ${IMAGE_NAME} > /dev/null

                    echo "Docker image verified successfully:"
                    docker image inspect ${IMAGE_NAME} \
                        --format='{{.RepoTags}}'
                '''
            }
        }

        stage('Deploy Kubernetes Resources') {
            steps {
                sh '''
                    echo "=== Applying Kubernetes Namespace ==="
                    kubectl apply -f k8s/namespace.yaml

                    echo "=== Applying PostgreSQL Resources ==="
                    kubectl apply -f k8s/postgres-pvc.yaml
                    kubectl apply -f k8s/postgres-deployment.yaml
                    kubectl apply -f k8s/postgres-service.yaml
                '''
            }
        }

        stage('Wait for PostgreSQL') {
            steps {
                sh '''
                    echo "=== Waiting for PostgreSQL ==="

                    kubectl rollout status \
                        deployment/postgres \
                        -n ${NAMESPACE} \
                        --timeout=120s
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    echo "=== Deploying Backend ==="

                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/backend-service.yaml

                    echo "=== Updating Backend Image ==="

                    kubectl set image deployment/backend \
                        backend=${IMAGE_NAME} \
                        -n ${NAMESPACE}

                    echo "=== Verifying Backend Image ==="

                    kubectl get deployment backend \
                        -n ${NAMESPACE} \
                        -o jsonpath='{.spec.template.spec.containers[0].image}'

                    echo
                '''
            }
        }

        stage('Wait for Backend') {
            steps {
                sh '''
                    echo "=== Waiting for Backend Rollout ==="

                    kubectl rollout status \
                        deployment/backend \
                        -n ${NAMESPACE} \
                        --timeout=120s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "=== Pods ==="
                    kubectl get pods -n ${NAMESPACE} -o wide

                    echo "=== Services ==="
                    kubectl get services -n ${NAMESPACE}

                    echo "=== Backend Deployment ==="
                    kubectl get deployment/backend -n ${NAMESPACE}

                    echo "=== Backend Image ==="
                    kubectl get deployment/backend \
                        -n ${NAMESPACE} \
                        -o jsonpath='{.spec.template.spec.containers[0].image}'

                    echo
                '''
            }
        }
    }

    post {
        success {
            echo '=========================================='
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY'
            echo '=========================================='
        }

        failure {
            echo '=========================================='
            echo 'CI/CD PIPELINE FAILED'
            echo '=========================================='

            sh '''
                echo "=== Pod Status ==="
                kubectl get pods -n ${NAMESPACE} -o wide || true

                echo "=== Backend Logs ==="
                kubectl logs deployment/backend \
                    -n ${NAMESPACE} \
                    --tail=100 || true
            '''
        }

        always {
            echo "Pipeline completed with status: ${currentBuild.currentResult}"
        }
    }
}