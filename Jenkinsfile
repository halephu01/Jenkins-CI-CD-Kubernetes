pipeline {
    environment {
        DOCKER_REGISTRY = "tuilakhanh"
        BUILD_TAG = "v${BUILD_NUMBER}-${GIT_COMMIT[0..7]}"
        DOCKER_CREDENTIALS_ID = 'dockercerd'
        KUBE_CONFIG_ID = 'k8s-config'
        KUBE_CLUSTER_NAME = 'minikube'
        KUBE_CONTEXT_NAME = 'minikube'
        KUBE_SERVER_URL = 'https://192.168.49.2:8443'
        REPORT_DIR = 'reports'
        SONAR_PROJECT_BASE_DIR = '.'
        SONAR_SCANNER_OPTS = '-Xmx2048m'
    }
    
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    sh 'mvn test clean'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('sq1') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${env.JOB_NAME} \
                            -Dsonar.projectName=${env.JOB_NAME} \
                            -Dsonar.sources=. \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.qualitygate.wait=false
                        """
                    }
                    
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate projectKey: env.JOB_NAME
                        if (qg.status != 'OK') {
                            error "Quality gate failed: ${qg.status}"
                        }
                        echo "Quality gate passed"
                    }
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("", DOCKER_CREDENTIALS_ID) {
                        def serviceImage = docker.build("${DOCKER_REGISTRY}/spring-boot-app:${BUILD_TAG}")
                        serviceImage.push()
                        serviceImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(clusterName: KUBE_CLUSTER_NAME, contextName: KUBE_CONTEXT_NAME, credentialsId: KUBE_CONFIG_ID, serverUrl: KUBE_SERVER_URL) {
                        sh """
                            kubectl apply -f k8s/base/namespace.yml || true
                            kubectl apply -f k8s/base/config.yaml || true
                            kubectl apply -f k8s/base/services.yaml || true
                            
                            kubectl set image deployment/spring-boot-app spring-boot-app=${DOCKER_REGISTRY}/spring-boot-app:${BUILD_TAG}
                            
                            kubectl rollout status deployment/spring-boot-app
                        """
                    }
                }
            }
        }
        
        stage('Verify Deployments') {
            steps {
                verifyDeployments()
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

// Helper functions
def verifyDeployments() {
    withKubeConfig(clusterName: KUBE_CLUSTER_NAME, contextName: KUBE_CONTEXT_NAME, credentialsId: KUBE_CONFIG_ID, serverUrl: KUBE_SERVER_URL) {
        sh '''
            echo "Services Status:"
            kubectl get svc -n microservices
            
            echo "\nPods Status:"
            kubectl get pods -n microservices
            
            echo "\nDeployments Status:"
            kubectl get deployments -n microservices
        '''
    }
}