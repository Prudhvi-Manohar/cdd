pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image with BUILD_ID for versioning
                    bat 'docker build -t flask-app:%BUILD_ID% .'
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    // Clean up any existing test container first
                    bat 'docker stop flask-test || echo "Container not running"'
                    bat 'docker rm flask-test || echo "Container not found"'
                    
                    // Start the container for testing
                    bat 'docker run -d --name flask-test -p 5001:5000 flask-app:%BUILD_ID%'
                    
                    // Wait for app to start - using ping as a delay technique in Windows
                    bat 'ping -n 6 127.0.0.1 > nul'
                    
                    // Verify the application is running by making a request
                    bat '''
                        powershell -command "try { $response = Invoke-WebRequest -Uri http://localhost:5001 -UseBasicParsing; if($response.StatusCode -eq 200) { exit 0 } else { exit 1 } } catch { exit 1 }"
                    '''
                }
            }
            post {
                always {
                    // Clean up test container whether the test passed or failed
                    bat 'docker stop flask-test || echo "Could not stop container"'
                    bat 'docker rm flask-test || echo "Could not remove container"'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Clean up any existing production container
                    bat 'docker stop flask-production || echo "Container not running"'
                    bat 'docker rm flask-production || echo "Container not found"'
                    
                    // Deploy new version
                    bat 'docker run -d --name flask-production -p 5000:5000 flask-app:%BUILD_ID%'
                    
                    // Verify deployment
                    bat 'ping -n 6 127.0.0.1 > nul'
                    bat '''
                        powershell -command "try { $response = Invoke-WebRequest -Uri http://localhost:5000 -UseBasicParsing; if($response.StatusCode -eq 200) { Write-Host 'Deployment verified!' } else { Write-Host 'Deployment verification failed!'; exit 1 } } catch { Write-Host 'Failed to connect to application!'; exit 1 }"
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
            
            // Additional cleanup in case of failure
            bat 'docker stop flask-test || echo "Container not running"'
            bat 'docker rm flask-test || echo "Container not found"'
        }
    }
}
