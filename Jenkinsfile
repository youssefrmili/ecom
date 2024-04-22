def microserviceFolders = ['ecomm-cart', 'ecomm-order', 'ecomm-product', 'ecomm-web']
pipeline {
    agent any

    // Define the microservice folder names
 

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
                            sh 'mvn test > f1'

                            // Save test report as a file
                           
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

                                // Save SonarQube report as a file
                                sh 'cp target/sonar/report-task.txt sonar_report.txt'
                            }
                        }
                    }
                }
            }
        }
        
        stage('Slack Notification') {
            steps {
                script {
                    // Upload SonarQube report to Slack
                    slackSend channel: 'test', message: 'SonarQube Analysis Report', teamDomain: 'Sofiatech PFE Interns ', username: 'jenkins', file: 'sonar_report.txt'
                    
                    // Upload test report to Slack
                    slackSend channel: 'test', message: 'Test Report', teamDomain: 'Sofiatech PFE Interns ', username: 'jenkins', file: 'test_report.xml'
                }
            }
        }
    }
}
