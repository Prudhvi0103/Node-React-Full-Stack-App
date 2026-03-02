pipeline {
    agent any
    tools{
        nodejs 'node16'
            sonar-scanner 'sonar-scanner'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {

      stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }

      
           stage('Git-Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/nahidkishore/Node-React-Full-Stack-App.git'
            }
        }
        

        

         stage('SonarQube analysis') {
        steps {
            withSonarQubeEnv('SonarQube') {  // server name
                sh 'sonar-scanner'   // or sonar-scanner.bat on Windows
            }
        }
    }

        stage("SonarQube Quality Gate"){
       
             steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
         }

     }
     
       
      

      stage("OWASP Dependency Check"){
           steps{
                dependencyCheck additionalArguments: '--scan ./' , odcInstallation: 'DP-Check'
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


        stage("Build and Push to Docker Hub"){
               steps{
                   
                echo 'login into docker hub and pushing image....'
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
                     sh "docker build -t node-full-stack-app:latest -f backend/Dockerfile ."
                     sh "docker tag node-full-stack-app ${env.dockerHubUser}/node-full-stack-app:latest"
                     sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                     sh "docker push ${env.dockerHubUser}/node-full-stack-app:latest"


               }
           }
         }


         stage("TRIVY Docker Scan"){
            steps{

                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]){
                     
                    sh "trivy image ${env.dockerHubUser}/node-full-stack-app:latest" 

               }
                
            }
        }

         stage("Deploy to Docker Container"){
            steps{
                sh " docker run -d --name node-full-stack-app -p 4000:4000 nahid0002/node-full-stack-app:latest "
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                script {
                   
                    withKubeConfig([credentialsId: 'K8s', serverUrl: '']) {
                        sh ('kubectl apply -f deployment.yaml')
                    }
                }
            }
        }

        stage('Clean up Containers') {   //if container runs it will stop and remove this block
          steps {
           script {
             try {
                sh 'docker stop node-full-stack-app'
                sh 'docker rm node-full-stack-app'
                } catch (Exception e) {
                  echo "Container node-full-stack-app not found, moving to next stage"  
                }
            }
          }
        }
     
    }


 post {
            always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'nahidkishore99@gmail.com',
            attachmentsPattern: 'trivy.txt'
        }
        }

}
