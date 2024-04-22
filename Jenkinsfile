    // Define the microservice folder names
def microserviceFolders = ['ecomm-cart', 'ecomm-order', 'ecomm-product', 'ecomm-web']
pipeline {
    agent any

    stages {
        stage('SCM Checkout') {
            steps {
                script {
                    CI_ERROR = "Failed while checking out SCM"
                    // Code for SCM Checkout
                    checkout([
                        $class: 'GitSCM', 
                        branches: [[name: 'feature']], 
                        userRemoteConfigs: [[url: 'https://github.com/youssefrmili/ecom.git']]
                    ])
                }
            }
        }

        stage('Build Application') {
            steps {
                script {
                    CI_ERROR = "Failed while building application"
                    // Code for building application
                    for (def folder in microserviceFolders) {
                        dir(folder) {
                            sh 'mvn clean install'
                        }
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    CI_ERROR = "Failed while deploying application"
                    // Code for deploying application
                }
            }
        }

        stage('Test Microservices') {
            steps {
                script {
                    CI_ERROR = "Failed while testing microservices"
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
                    CI_ERROR = "Failed during SonarQube analysis"
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
        
        stage('Slack Notification') {
            steps {
                script {
                    // Call the function to send Slack notification
                    sendSlackNotifcation()
                }
            }
        }
    }
    
    post {
        always {
            script {
                CONSOLE_LOG = "${env.BUILD_URL}/console"
                BUILD_STATUS = currentBuild.currentResult
                if (currentBuild.currentResult == 'SUCCESS') {
                    CI_ERROR = "NA"
                }
            }
            sh """
                TODAY=`date +"%b %d"`
                sed -i "s/%%JOBNAME%%/${env.JOB_NAME}/g" report.html
                sed -i "s/%%BUILDNO%%/${env.BUILD_NUMBER}/g" report.html
                sed -i "s/%%DATE%%/\${TODAY}/g" report.html
                sed -i "s/%%BUILD_STATUS%%/${BUILD_STATUS}/g" report.html
                sed -i "s/%%ERROR%%/${CI_ERROR}/g" report.html
                sed -i "s|%%CONSOLE_LOG%%|${CONSOLE_LOG}|g" report.html
            """
            publishHTML(target:[
                allowMissing: true,
                alwaysLinkToLastBuild: true, 
                keepAll: true, 
                reportDir: "${WORKSPACE}", 
                reportFiles: 'report.html', 
                reportName: 'CI-Build-HTML-Report', 
                reportTitles: 'CI-Build-HTML-Report'
            ])
        }
    }
}

def sendSlackNotifcation() {
    if ( currentBuild.currentResult == "SUCCESS" ) {
        buildSummary = "Job:  ${env.JOB_NAME}\n Status: *SUCCESS*\n Build Report: ${env.BUILD_URL}CI-Build-HTML-Report"

        slackSend color : "good", message: "${buildSummary}", channel: 'test'
    } else {
        buildSummary = "Job:  ${env.JOB_NAME}\n Status: *FAILURE*\n Error description: *${CI_ERROR}* \nBuild Report :${env.BUILD_URL}CI-Build-HTML-Report"
        slackSend color : "danger", message: "${buildSummary}", channel: 'test'
    }
}
