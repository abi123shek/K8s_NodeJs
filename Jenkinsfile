pipeline {
    
    agent any
     environment{
        SONAR_HOME = tool "Sonar"
    }
    stages {
        
        stage("Code"){
            steps{
                git url: "https://github.com/mandgepratik/node-todo-cicd.git" , branch: "master"
                echo "Code Cloned Successfully"
            }
        }
        stage("SonarQube Analysis"){
            steps{
               withSonarQubeEnv("Sonar"){
                   sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=nodetodo -Dsonar.projectKey=nodetodo -X"
               }
            }
        }
        stage("SonarQube Quality Gates"){
            steps{
               timeout(time: 1, unit: "MINUTES"){
                   waitForQualityGate abortPipeline: false
               }
            }
        }
        stage("Build"){
            steps{
                sh 'docker build -t node-app:latest .'
                echo "Code Built Successfully"
            }
        }
        stage("Test"){
            steps{
                echo "Build Tested Successfully"
            }
        }
        stage("Push to Private Docker Hub Repo"){
            steps{
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerPass",usernameVariable:"dockerUser")]){
                sh "docker login -u ${env.dockerUser} -p ${env.dockerPass}"
                sh "docker tag node-app:latest ${env.dockerUser}/node-app:latest"
                sh "docker push ${env.dockerUser}/node-app:latest"
                }
                
            }
        }
        stage("Sonar"){
            steps{
                echo "Build Tested Successfully"
            }
        }
        stage("OWASP"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Trivy"){
            steps{
                sh "trivy image node-app"
                echo "Image Scanned Successfully"
            }
        }
        stage("Deploy"){
            steps{
                sh "docker-compose up -d"
                echo "App Deployed Successfully"
            }
        }
    }
}
