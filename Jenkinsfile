@Library("Shared") _
pipeline{
    agent {label "dev"};
    

    
    stages{
        stage("Code Clone"){
            steps{
                script{
                    clone("https://github.com/sopatel14/two-tier-flask-app.git", "master") 
                }
            }
        }
        stage("Trivy File system sacn"){
            steps{
                script{
                    trivy_fs()
                    
                }
                
    
                
            }

        }
        
        stage("Build"){
            steps{
                sh "docker build -t two-tier-flask-app ." 
            }
        }
        
        stage("Push to Docker Hub"){
            steps{
               script{

                   docker_push("dockerHubCreds","two-tier-flask-app")
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

post {
    success{
        emailext(
        subject: "Build successful",
        body: "Good news: your build was successful!!",
        to: 'souravpatel65@gmail.com'
            )
        
        
    }
    failure{
        emailext(
        subject: "Build failed",
        body: "Bad mews: your build was failed",
        to: 'souravpatel65@gmail.com'
            
            
            )
        
        
        
        
        
        
    }
    
}
}

