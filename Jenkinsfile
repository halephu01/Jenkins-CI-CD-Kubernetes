pipeline {
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
         
        USER_SERVICE_IMAGE = 'halephu01/user-service'
        FRIEND_SERVICE_IMAGE = 'halephu01/friend-service'
        AGGREGATE_SERVICE_IMAGE = 'halephu01/aggregate-service'

        KUBE_CLUSTER_NAME = 'minikube'
        KUBE_CONTEXT_NAME = 'minikube'
        KUBE_SERVER_URL = 'https://192.168.49.2:51792'
        SONAR_PROJECT_BASE_DIR = '.'

        VERSION = "${BUILD_NUMBER}"
        SONAR_TOKEN = credentials('scan')
        SONAR_PROJECT_KEY = 'microservices-project'

    }
    
    agent any
    
    tools {
        maven 'Maven 3.8.6'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/halephu01/Jenkins-CI-CD-Kubernetes.git',
                    credentialsId: 'github-credentials'
            }
        }    

        // stage('Build') {
        //     steps {
        //         script {
        //             sh 'mvn clean package -DskipTests'
        //         }
        //     }
        // }
        
        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             def scannerHome = tool 'SonarScanner'
        //             def services = ['user-service', 'friend-service', 'aggregate-service']

        //             withSonarQubeEnv('sonar') {
        //                 services.each { service ->
        //                     dir(service) {
        //                         sh """
        //                             ${scannerHome}/bin/sonar-scanner \
        //                             -Dsonar.projectKey=${service} \
        //                             -Dsonar.projectName=${service} \
        //                             -Dsonar.sources=. \
        //                             -Dsonar.java.binaries=target/classes \
        //                         """
        //                     }
        //                 }
        //             }
        //         }
        //     }
        // }
        
        // stage('Build and Push Docker Images') {
        //     steps {
        //         script {
        //             // Định nghĩa services
        //             def services = ['user-service', 'friend-service', 'aggregate-service']
                    
        //             services.each { service ->
        //                 echo "Building ${service} Docker image..."
        //                 try {
        //                     // Build với tên image chuẩn
        //                     sh """
        //                         docker build -t ${service} -f ${service}/Dockerfile .
        //                     """
                            
        //                     // Tag image với version
        //                     sh """
        //                         docker tag ${service} halephu01/${service}:${BUILD_NUMBER}
        //                         docker tag ${service} halephu01/${service}:latest
        //                     """
                            
        //                     echo "Successfully built ${service} image"
        //                 } catch (Exception e) {
        //                     echo "Error building ${service}: ${e.message}"
        //                     throw e
        //                 }
        //             }
        //         }
        //     }
        // }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(clusterName: KUBE_CLUSTER_NAME, contextName: KUBE_CONTEXT_NAME, serverUrl: KUBE_SERVER_URL) {
                        sh """

                            minikube start

                            kubectl apply -k k8s/base
                            
                            kubectl apply -k k8s/base/services/aggregate-service
                            kubectl apply -k k8s/base/services/friend-service
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
    withKubeConfig(clusterName: KUBE_CLUSTER_NAME, contextName: KUBE_CONTEXT_NAME, serverUrl: KUBE_SERVER_URL) {
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