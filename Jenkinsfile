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
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format ALL', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt || true"  // non-fatal if trivy missing
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
        
        // ... rest of your stages (TRIVY image, deploy, cleanup) remain the same
    }
    
    post {
        always {
            emailext(
                attachLog: true,
                subject: "${currentBuild.result} - Node React App #${BUILD_NUMBER}",
                body: "Build: ${BUILD_URL}\nStatus: ${currentBuild.result}",
                to: 'nahidkishore99@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
