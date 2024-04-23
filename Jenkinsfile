def microserviceFolders = ['ecomm-cart', 'ecomm-order', 'ecomm-product', 'ecomm-web']

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout the main repository
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*']], 
                    userRemoteConfigs: [[url: 'https://github.com/youssefrmili/ecom.git']]
                ])
            }
        }

        stage('Build Microservices') {
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def folder in microserviceFolders) {
                        // Navigate into the microservice folder
                        dir(folder) {
                            // Build the microservice
                            sh 'mvn clean install'
                        }
                    }
                }
            }
        }

        stage('Test Microservices') {
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def folder in microserviceFolders) {
                        // Navigate into the microservice folder
                        dir(folder) {
                            // Test the microservice
                            sh 'mvn test'
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def folder in microserviceFolders) {
                        // Navigate into the microservice folder
                        dir(folder) {
                            // Execute SAST with SonarQube
                            withSonarQubeEnv(credentialsId: 'sonarqube-id') {
                                sh 'mvn sonar:sonar'
                                sh 'cat target/sonar/report-task.txt'
                            }
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def folder in microserviceFolders) {
                        // Navigate into the microservice folder
                        dir(folder) {
                            // Build Docker image
                            sh "docker build -t youssefrm/${folder}:latest ."
                        }
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            environment {
                DOCKER_CREDENTIALS = credentials('c6ea3088-f7ad-4b7a-a9f6-3614f2de2f25')
            }
            steps {
                script {
                    // Iterate over each microservice folder
                    for (def folder in microserviceFolders) {
                        // Docker login
                        sh "docker login -u $DOCKER_CREDENTIALS_USR -p $DOCKER_CREDENTIALS_PSW"
                        // Push Docker image
                        sh "docker push youssefrm/${folder}:latest"
                    }
                }
            }
        }
    }
}
