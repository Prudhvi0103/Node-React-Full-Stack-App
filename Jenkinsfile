pipeline {
    agent any
    
    tools {
        nodejs 'node16'   // Make sure this name exists in Global Tool Configuration
    }
    
    environment {
        // Removed SCANNER_HOME (was SonarQube related)
        // You can keep other env vars if needed later
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
                    sh "docker build -t node-full-stack-app:latest -f backend/Dockerfile ."
                    sh "docker tag node-full-stack-app:latest ${env.dockerHubUser}/node-full-stack-app:latest"
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh "docker push ${env.dockerHubUser}/node-full-stack-app:latest"
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
                    sh "trivy image --exit-code 0 --no-progress ${env.dockerHubUser}/node-full-stack-app:latest"
                }
            }
        }
        
        stage('Deploy to Docker Container') {
            steps {
                // Stop and remove old container if exists (safe)
                sh 'docker stop node-full-stack-app || true'
                sh 'docker rm node-full-stack-app || true'
                
                // Run new container
                sh "docker run -d --name node-full-stack-app -p 4000:4000 ${env.dockerHubUser}/node-full-stack-app:latest"
            }
        }
        
        // Optional: Kubernetes stage – keep only if you have kubeconfig ready
        // stage('Deploy To Kubernetes') {
        //     steps {
        //         script {
        //             withKubeConfig([credentialsId: 'K8s', serverUrl: '']) {
        //                 sh 'kubectl apply -f deployment.yaml'
        //             }
        //         }
        //     }
        // }
        
        stage('Clean up Containers') {
            steps {
                script {
                    try {
                        sh 'docker stop node-full-stack-app || true'
                        sh 'docker rm node-full-stack-app || true'
                    } catch (Exception e) {
                        echo "Container cleanup: nothing to remove or already gone"
                    }
                }
            }
        }
    }
    
    post {
        always {
            emailext(
                attachLog: true,
                subject: "'${currentBuild.result}' - Node React Full Stack App",
                body: """Project: ${env.JOB_NAME}<br/>
Build Number: ${env.BUILD_NUMBER}<br/>
URL: ${env.BUILD_URL}<br/>
Status: ${currentBuild.result}""",
                to: 'nahidkishore99@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
