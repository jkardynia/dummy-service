pipeline {
    agent any

    tools {
        maven 'default-maven'
        jdk 'jdk21'
    }

    environment {
        DUMMY_APP = 'dummy-0.0.1-SNAPSHOT.jar' // Replace with your artifact name
    }


    stages {
        stage('Checkout Application') {
            steps {
                echo "application checkout"

                git branch: 'main',
                    credentialsId: 'jjarekk-GitHub',
                    url: 'https://github.com/jkardynia/dummy-service.git'
            }

        }

        stage('Build and Deploy Application') {
            steps {
                // Example: Build Docker container and deploy to staging environment
                echo "building application"
                sh 'java -version'
                sh 'mvn clean install'

                sh "nohup java -jar target/${env.DUMMY_APP} &"
            }
        }

        stage('Health Check') {
            steps {
                script {
            timeout(time: 5, unit: 'MINUTES') {
                waitUntil {
                    try {
                        def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8081/hello", returnStdout: true).trim()
                        echo "Response: ${response}"
                        return response == '200'
                    } catch (Exception e) {
                        echo "Failed to connect, retrying..."
                        return false
                    }
                }
            }
            echo 'Health check passed'
        }
            }
        }


        stage('Run E2E Tests') {
            steps {
                script {
                // Assuming tests are triggered via npm
                    echo "running tests"
                    // Replace 'your-existing-job-name' with the name of the job you wish to build
                    def externalJob = build job: 'Rooam/e2e-tests', parameters: [string(name: 'testsBranchName', value: 'main')], wait: true
                    // You can access properties like externalJob.result or externalJob.buildNumber
                    echo "Status: ${externalJob.result}"
                }
            }
        }
    }
    post {
            always {
                echo 'Cleaning up and archiving artifacts'
                sh "pkill -f ${env.DUMMY_APP}" // Ensure app is stopped even if build fails
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                junit '**/target/surefire-reports/TEST-*.xml' // Adjust path as necessary for your project
            }
            success {
                echo 'E2E tests passed.'
            }
            failure {
                echo 'E2E tests failed.'
                // Optionally, add notifications or other error handling
            }
        }
}