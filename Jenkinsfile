pipeline {
    agent any
    
    tools {
        nodejs 'node16'   // Confirm this name exists in Global Tool Configuration
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git-Checkout') {
            steps {
                git branch: 'main', 
                    changelog: false, 
                    poll: false, 
                    url: 'https://github.com/nahidkishore/Node-React-Full-Stack-App.git'
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', 
                                 odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }
        
        stage('Build and Push to Docker Hub') {
            steps {
                echo 'Building and pushing Docker image...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerHub', 
                    passwordVariable: 'dockerHubPassword', 
                    usernameVariable: 'dockerHubUser'
                )]) {
                    sh "docker build -t ${dockerHubUser}/node-full-stack-app:latest -f backend/Dockerfile ."
                    sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
                    sh "docker push ${dockerHubUser}/node-full-stack-app:latest"
                }
            }
        }
        
        stage('TRIVY Docker Image Scan') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerHub', 
                    passwordVariable: 'dockerHubPassword', 
                    usernameVariable: 'dockerHubUser'
                )]) {
                    sh "trivy image --exit-code 0 --no-progress ${dockerHubUser}/node-full-stack-app:latest"
                }
            }
        }
        
        stage('Deploy to Docker Container') {
            steps {
                // Safe stop/remove if exists
                sh 'docker stop node-full-stack-app || true'
                sh 'docker rm node-full-stack-app || true'
                
                sh "docker run -d --name node-full-stack-app -p 4000:4000 ${dockerHubUser}/node-full-stack-app:latest"
            }
        }
        
        stage('Clean up Containers') {
            steps {
                script {
                    try {
                        sh 'docker stop node-full-stack-app || true'
                        sh 'docker rm node-full-stack-app || true'
                    } catch (Exception e) {
                        echo "No container to clean up"
                    }
                }
            }
        }
    }
    
    post {
        always {
            emailext(
                attachLog: true,
                subject: "${currentBuild.result} - Node React Full Stack Build #${BUILD_NUMBER}",
                body: """Project: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Status: ${currentBuild.result}
URL: ${BUILD_URL}""",
                to: 'nahidkishore99@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
