def microservices = ['ecomm-cart', 'ecomm-product', 'ecomm-order', 'ecomm-web']

pipeline {
    agent any

    stages {
        stage('Checkout') {
            when {
                changeset '**/*'
            }
            steps {
                // Checkout the main repository using the new URL
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*']], 
                    userRemoteConfigs: [[url: 'https://github.com/youssefrmili/ecom.git']]
                ])
            }
        }

        stage('Check-Git-Secrets') {
            when {
                changeset '**/*'
            }
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            sh 'rm trufflehog || true'
                            sh 'docker run gesellix/trufflehog --json https://github.com/youssefrmili/ecom.git > trufflehog'
                            sh 'cat trufflehog'
                        }
                    }
                }
            }
        }

        stage('Source Composition Analysis') {
            when {
                changeset '**/*'
            }
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            sh 'rm owasp* || true'
                            sh 'wget "https://raw.githubusercontent.com/youssefrmili/Ecommerce-APP/test/owasp-dependency-check.sh"'
                            sh 'chmod +x owasp-dependency-check.sh'
                            sh 'bash owasp-dependency-check.sh'
                            sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
                        }
                    }
                }
            }
        }

        stage('Build') {
            when {
                changeset "**/${service}/**"
            }
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            sh 'mvn clean install'
                        }
                    }
                }
            }
        }

        stage('Unit Test') {
            when {
                changeset "**/${service}/**"
            }
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            sh 'mvn test'
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            when {
                changeset "**/${service}/**"
            }
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            withSonarQubeEnv(credentialsId: 'sonarqube-id') {
                                sh 'mvn sonar:sonar'
                                sh 'cat target/sonar/report-task.txt'
                            }
                        }
                    }
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                }
            }
        }

        stage('Docker Build') {
            when {
                changeset "**/${service}/**"
            }
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            sh "docker build -t youssefrm/${service}:latest ."
                        }
                    }
                }
            }
        }
    }
}
