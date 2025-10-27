pipeline{
    agent any
    
    stages{
        stage("Code Clone"){
            steps{
                git url: "https://github.com/sopatel14/two-tier-flask-app.git", branch: "master"
            }
        }
        
        stage("Build"){
            steps{
                sh "docker build -t two-tier-flask-app ." 
            }
        }
        
        stage("Push to Docker Hub"){
            steps{
                withCredentials([usernamePassword(
                    credentialsId:"dockerHubCreds",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                
                )]){
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker image tag two-tier-flask-app ${env.dockerHubUser}/two-tier-flask-app"
                sh "docker push ${env.dockerHubUser}/two-tier-flask-app:latest"
                
                }
                
            }
            
        }
        
        stage("Deploy"){
            steps{
                sh 'docker compose down || true'
                sh 'docker compose up -d'
            }
        }
    }
}
