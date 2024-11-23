pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        
        USER_SERVICE_IMAGE = 'halephu01/user-service'
        FRIEND_SERVICE_IMAGE = 'halephu01/friend-service'
        AGGREGATE_SERVICE_IMAGE = 'halephu01/aggregate-service'
        
        VERSION = "${BUILD_NUMBER}"
        
        SONAR_TOKEN = credentials('sonar')
        SONAR_PROJECT_KEY = 'microservices-project'
    }
    
    tools {
        maven 'Maven 3.8.6'
        jdk 'JDK 11'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/halephu01/Jenkins-CI-CD.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build Services') {
            parallel {
                stage('Build User Service') {
                    steps {
                        dir('user-service') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
                
                stage('Build Friend Service') {
                    steps {
                        dir('friend-service') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
                
                stage('Build Aggregate Service') {
                    steps {
                        dir('aggregate-service') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
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
        
        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    sh """
                        if docker image inspect halephu01/user-service:${BUILD_NUMBER} >/dev/null 2>&1; then
                            echo "Pushing user-service image..."
                            docker push halephu01/user-service:${BUILD_NUMBER}
                        else
                            echo "user-service image not found!"
                            exit 1
                        fi

                        if docker image inspect halephu01/friend-service:${BUILD_NUMBER} >/dev/null 2>&1; then
                            echo "Pushing friend-service image..."
                            docker push halephu01/friend-service:${BUILD_NUMBER}
                        else
                            echo "friend-service image not found!"
                            exit 1
                        fi

                        if docker image inspect halephu01/aggregate-service:${BUILD_NUMBER} >/dev/null 2>&1; then
                            echo "Pushing aggregate-service image..."
                            docker push halephu01/aggregate-service:${BUILD_NUMBER}
                        else
                            echo "aggregate-service image not found!"
                            exit 1
                        fi
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/user-service user-service=${USER_SERVICE_IMAGE}:${VERSION} --record
                        kubectl set image deployment/friend-service friend-service=${FRIEND_SERVICE_IMAGE}:${VERSION} --record
                        kubectl set image deployment/aggregate-service aggregate-service=${AGGREGATE_SERVICE_IMAGE}:${VERSION} --record
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed!'
        }
        always { 
            script {
                try {
                    sh """
                        docker-compose down || true
                        docker logout
                        
                        # Remove images
                        for service in ${env.DOCKER_IMAGES}; do
                            image="halephu01/\${service}:${BUILD_NUMBER}"
                            if docker image inspect \${image} >/dev/null 2>&1; then
                                echo "Removing image \${image}..."
                                docker rmi \${image} || true
                            fi
                        done
                    """
                } catch (Exception e) {
                    echo "Cleanup failed: ${e.message}"
                }
            }
        }
    }
} 