pipeline {
    environment {
        DOCKER_REGISTRY = "halephu01"
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
    
    tools {
        maven 'Maven 3.8.6'
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    def services = ['user-service', 'friend-service', 'aggregate-service']

                    withSonarQubeEnv('SonarScanner') {
                        services.each { service ->
                            dir(service) {
                                sh """
                                    ${scannerHome}/bin/sonar-scanner \
                                    -Dsonar.projectKey=${service} \
                                    -Dsonar.projectName=${service} \
                                    -Dsonar.sources=. \
                                    -Dsonar.java.binaries=target/classes \
                                """
                            }
                        }
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
                            kubectl apply -k k8s/base
                            
                            kubectl delete -k k8s/base/services/aggregate-service
                            kubectl delete -k k8s/base/services/friend-service
                            kubectl apply -k k8s/base/services/user-service                           
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