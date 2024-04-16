pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Clone the repository
                git 'https://github.com/youssefrmili/ecom.git'

                // Build each microservice individually
                dir('ecomm-cart') {
                    script {
                        // Build ecomm-cart microservice
                        sh 'mvn clean package'
                    }
                }

                dir('ecomm-db') {
                    script {
                        // Build ecomm-db microservice
                        sh 'mvn clean package'
                    }
                }

                dir('ecomm-gateway') {
                    script {
                        // Build ecomm-gateway microservice
                        sh 'mvn clean package'
                    }
                }

                // Repeat the above pattern for other microservices as needed
            }
        }
    }
}
