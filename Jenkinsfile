pipeline {
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
         
        USER_SERVICE_IMAGE = 'halephu01/user-service'
        FRIEND_SERVICE_IMAGE = 'halephu01/friend-service'
        AGGREGATE_SERVICE_IMAGE = 'halephu01/aggregate-service'
    
        KUBE_CONFIG_ID = 'minikube'
        KUBE_CLUSTER_NAME = 'minikube'
        KUBE_CONTEXT_NAME = 'minikube'
        KUBE_SERVER_URL = 'https:/192.168.58.2:8443'

        VERSION = "${BUILD_NUMBER}"
        SONAR_TOKEN = credentials('sonar')
    }


    agent any
    
    stages {    
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/halephu01/Jenkins-CI-CD-Kubernetes.git',
                    credentialsId: 'github-credentials'
            }
        }    

        // stage('Build Services') {
        //     parallel {
        //         stage('Build User Service') {
        //             steps {
        //                 dir('user-service') {
        //                     sh 'mvn clean package -DskipTests'
        //                 }
        //             }
        //         }
                
        //         stage('Build Friend Service') {
        //             steps {
        //                 dir('friend-service') {
        //                     sh 'mvn clean package -DskipTests'
        //                 }
        //             }
        //         }
                
        //         stage('Build Aggregate Service') {
        //             steps {
        //                 dir('aggregate-service') {
        //                     sh 'mvn clean package -DskipTests'
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             def scannerHome = tool 'sonar'
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
        
        // // stage('Build and Push Docker Images') {
        // //     steps {
        // //         script {
        // //             def services = ['user-service', 'friend-service', 'aggregate-service']
                    
        // //             services.each { service ->
        // //                 echo "Building ${service} Docker image..."
        // //                 try {
        // //                     sh """
        // //                         docker build -t ${service} -f ${service}/Dockerfile .
                        
        // //                         docker tag ${service} halephu01/${service}:latest
        // //                     """
                            
        // //                     echo "Successfully built ${service} image"
        // //                 } catch (Exception e) {
        // //                     echo "Error building ${service}: ${e.message}"
        // //                     throw e
        // //                 }
        // //             }
        // //         }
        // //     }
        // // }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(clusterName: KUBE_CLUSTER_NAME, contextName: KUBE_CONTEXT_NAME, credentialsId: KUBE_CONFIG_ID, serverUrl: KUBE_SERVER_URL) {
                        sh """
                            kubectl apply -k k8s/base
                            kubectl apply -k k8s/base/services/aggregate-service
                            kubectl apply -k k8s/base/services/user-service
                            kubectl apply -k k8s/base/services/friend-service
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}