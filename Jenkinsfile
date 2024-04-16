pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Clone the repository
                git 'https://github.com/youssefrmili/ecom.git'

                // Build ecomm-cart microservice
                dir('ecomm-cart') {
                    script {
                        sh 'mvn clean package'
                    }
                }

                // Repeat the above pattern for other microservices as needed
            }
        }
    }
}
