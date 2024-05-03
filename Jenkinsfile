def microservices = ['ecomm-cart', 'ecomm-product', 'ecomm-order', 'ecomm-web']

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                // Checkout the main repository
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*']],
                    userRemoteConfigs: [[url: 'https://github.com/youssefrmili/Ecommerce-APP.git']]
                ])
            }
        }

        stage('Validation') {
            steps {
                script {
                    dir('service') {
                        def strCount = sh(returnStdout: true, script: "git diff --name-only ${env.GIT_COMMIT} ${env.GIT_PREVIOUS_SUCCESSFUL_COMMIT} | grep service | wc -l").trim()
                        if (strCount == "0") {
                            echo "Skipping build as no files updated in the service module"
                            CONTINUE_BUILD = false
                        } else {
                            echo "Changes found in the service module"
                        }
                    }
                }
            }
        }

        stage('Check-Git-Secrets') {
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            sh 'rm trufflehog || true'
                            sh 'docker run gesellix/trufflehog --json https://github.com/youssefrmili/Ecommerce-APP.git > trufflehog'
                            sh 'cat trufflehog'
                        }
                    }
                }
            }
        }

        stage('Source Composition Analysis') {
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
            // Add condition to only run if the branch is 'master' or 'test'
            when {
                anyOf {
                    branch 'master'
                    branch 'test'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    for (def service in microservices) {
                        dir(service) {
                            sh "docker build -t youssefrm/${service}:latest ."
                        }
                    }
                }
            }
            when {
                anyOf {
                    branch 'master'
                    branch 'test'
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    for (def service in microservices) {
                        sh "docker run --rm -v /home/youssef/.cache:/root/.cache/ aquasec/trivy image --scanners vuln --timeout 15m youssefrm/${service}:latest > trivy.txt"
                    }
                }
            }
            when {
                anyOf {
                    branch 'master'
                    branch 'test'
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    for (def service in microservices) {
                        sh "docker push youssefrm/${service}:latest"
                    }
                }
            }
            when {
                anyOf {
                    branch 'master'
                    branch 'test'
                }
            }
        }
    }
}
