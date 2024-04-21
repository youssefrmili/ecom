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

        stage('Build and Test Microservices') {
            steps {
                script {
                    // List all directories in the repository
                    def microserviceFolders = sh(script: 'ls -d */', returnStdout: true).trim().split()

                    // Select the first seven folders
                    def selectedFolders = microserviceFolders.take(7)

                    // Iterate over each selected folder
                    for (def folder in selectedFolders) {
                        // Navigate into the microservice folder
                        dir(folder) {
                            // Build the microservice
                            sh 'mvn clean install'

                            // Test the microservice
                            sh 'mvn test'
                        }
                    }
                }
            }
        }

        stage('SAST and Quality Gate') {
            steps {
                script {
                    // List all directories in the repository
                    def microserviceFolders = sh(script: 'ls -d */', returnStdout: true).trim().split()

                    // Select the first seven folders
                    def selectedFolders = microserviceFolders.take(7)

                    // Iterate over each selected folder
                    for (def folder in selectedFolders) {
                        // Navigate into the microservice folder
                        dir(folder) {
                            // Execute SAST with SonarQube
                            withSonarQubeEnv(credentialsId: 'sonarqube-id') {
                                sh 'mvn sonar:sonar'
                                sh 'cat target/sonar/report-task.txt'
                            }

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
