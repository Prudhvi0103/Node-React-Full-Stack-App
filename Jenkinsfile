pipeline {
    agent any
    
    tools {
        nodejs 'node16'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git-Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nahidkishore/Node-React-Full-Stack-App.git'
            }
        }
        
        // Temporarily disabled until OWASP plugin is installed
        // stage('OWASP Dependency Check') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --format ALL', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh 'trivy fs . > trivyfs.txt || echo "Trivy scan failed (non-fatal)"'
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
                withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "docker build -t ${USER}/node-full-stack-app:latest -f backend/Dockerfile ."
                    sh "docker login -u ${USER} -p ${PASS}"
                    sh "docker push ${USER}/node-full-stack-app:latest"
                }
            }
        }
        
        stage('TRIVY Docker Image Scan') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "trivy image --exit-code 0 --no-progress ${USER}/node-full-stack-app:latest"
                }
            }
        }
        
        stage('Deploy to Docker Container') {
            steps {
                sh 'docker stop node-full-stack-app || true'
                sh 'docker rm node-full-stack-app || true'
                sh "docker run -d --name node-full-stack-app -p 4000:4000 ${USER}/node-full-stack-app:latest"
            }
        }
        
        stage('Clean up Containers') {
            steps {
                sh 'docker stop node-full-stack-app || true'
                sh 'docker rm node-full-stack-app || true'
            }
        }
    }
    
    post {
        always {
            emailext(
                attachLog: true,
                subject: "${currentBuild.result} - Node React App #${BUILD_NUMBER}",
                body: "Build URL: ${BUILD_URL}\nStatus: ${currentBuild.result}",
                to: 'nahidkishore99@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
