pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout the main repository
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: 'feature']], 
                    userRemoteConfigs: [[url: 'https://github.com/youssefrmili/ecom.git']]
                ])
            }
        }

        stage('Build Microservices') {
            steps {
                script {
                    // Define the microservice folder names
                    def microserviceFolders = ['ecomm-cart', 'ecomm-order', 'ecomm-product', 'ecomm-web']

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
                    // Define the microservice folder names again
                    def microserviceFolders = ['ecomm-cart', 'ecomm-order', 'ecomm-product', 'ecomm-web']

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

        stage('SonarQube Analysis & Quality Gate') {
            steps {
                script {
                    // Define the microservice folder names again
                    def microserviceFolders = ['ecomm-cart', 'ecomm-order', 'ecomm-product', 'ecomm-web']

                    // Iterate over each microservice folder
                    for (def folder in microserviceFolders) {
                        // Navigate into the microservice folder
                        dir(folder) {
                            // Execute SAST with SonarQube
                            withSonarQubeEnv(credentialsId: 'sonarqube-id' installationName: 'sonarqube') {
                                sh 'mvn sonar:sonar'
                                sh 'cat target/sonar/report-task.txt'

                                // Run quality gate
                                timeout(time: 1, unit: 'MINUTES') {
                                    waitForQualityGate abortPipeline: true
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
