pipeline {
     agent { label "kubernetes_server"}
     
       environment {
        IMAGE_NAME = "kubernetes-nodejs"
        IMAGE_TAG = "latest"
        DOCKER_HUB_REPO = "${env.DockerConfigUser}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    
     
     stages {
         stage("code") {
             steps {
                 git url: "https://github.com/abi123shek/K8s_NodeJs.git", branch: "main"
                 echo "Code cloned successfully"
             }
         }
         
           stage('SonarQube Code Analysis') {
            steps {
                dir("${WORKSPACE}"){
               
                 script {
                    def scannerHome = tool name: 'sonar7.0', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('sonar') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -D sonar.projectVersion=1.0-SNAPSHOT \
                            -D sonar.projectName=Kubernetes-nodejs_Project \
                            -D sonar.projectKey=Kubernetes-nodejs_NOTES \
                            -D sonar.sources=. \
                            -D sonar.host.url=http://54.88.4.86:9000"
                        }
                    }
                }
            }
        }
            
         stage("build") {
             steps {
                 sh 'ls -l'
               
                 sh 'docker build . -t kubernetes-nodejs:latest'
                 echo "Code build successfully"
             }
         }
         
          stage("Trivy Security Scan") {
            steps {
                echo "Running Trivy security scan on Docker image"
                sh '''
                    trivy image --exit-code 0 --format json --output trivy-report.json ${IMAGE_NAME}:${IMAGE_TAG}
                    trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG} || echo "Trivy scan found vulnerabilities!"
                '''
                
                archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
                echo "Trivy security scan completed"
            }
        }
        
         
         
          stage("push to dockerhub") {
             steps {
                  echo "Connecting to docker hub"
                  withCredentials([usernamePassword('credentialsId':"DockerConfig", 
                  passwordVariable:"DockerConfigpass", usernameVariable:"DockerConfigUser")]){
                      
                 sh "docker login -u ${env.DockerConfigUser} -p ${env.DockerConfigpass}" 
                 sh "docker image tag kubernetes-nodejs:latest  ${env.DockerConfigUser}/kubernetes-nodejs:latest"
                 sh "docker push ${env.DockerConfigUser}/kubernetes-nodejs:latest"
                 echo "image push successfully"
                  }
             }
         }
//       stage("Trigger ArgoCD Deployment") {
//       steps {
//         withCredentials([usernamePassword(credentialsId: 'argocd-credentials', 
//         passwordVariable: 'ARGO_PASS', usernameVariable: 'ARGO_USER')]) {
//             sh """
//                 argocd login 54.88.4.86:30383 --username \${ARGO_USER} --password \${ARGO_PASS} --insecure
//                 argocd app sync kubernetes-nodejs
//             """
//         }
//         echo "ArgoCD Deployment Triggered"
//     }
// }
          
          stage('Stop Running Containers') {
            steps {
                sh 'docker stop $(docker ps -q) || true'
                sh 'docker rm -f $(docker ps -aq) || true'
                echo 'Stopped and removed existing containers'
            }
        }

         stage("deploy") {
             steps {
                 sh "docker run -p 8000:8000 -d kubernetes-nodejs:latest" 
                 echo "Node.js application deployed successfully"
             }
         }
     }
     
      post {
        success {
            emailext (
                to: 'abishekchamlagai123@gmail.com',
                subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Good news! The build was successful.\n\nCheck details: ${env.BUILD_URL}"
            )
        }
        failure {
            emailext (
                to: 'abishekchamlagai123@gmail.com',
                subject: "Jenkins Build FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Oops! The build failed.\n\nCheck logs: ${env.BUILD_URL}"
            )
        }
    }
 }
