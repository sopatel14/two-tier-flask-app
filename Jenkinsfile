@Library("Shared") _
pipeline {
    agent { label "dev" }

    stages {
        stage("Code Clone") {
            steps {
                script {
                    clone("https://github.com/sopatel14/two-tier-flask-app.git", "master")
                }
            }
        }

        stage("Trivy File System Scan") {
            steps {
                script {
                    trivy_fs()
                }
            }
        }

        stage("Build") {
            steps {
                sh "docker build -t two-tier-flask-app ."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                script {
                    docker_push("dockerHubCreds", "two-tier-flask-app")
                }
            }
        }

        stage("Deploy") {
            steps {
                dir("${WORKSPACE}") {
                    echo "🛠️ Deploying the Flask + MySQL stack..."
                    
                    // Stop previous containers
                    sh "docker compose down || true"

                    // Pull the latest image
                    sh "docker compose pull flask-app || true"

                    // Start both containers (MySQL + Flask)
                    sh "docker compose up -d --build"

                    // ✅ Wait for MySQL to become healthy before Flask connects
                    sh '''
                    echo "⏳ Waiting for MySQL to become healthy..."
                    for i in {1..10}; do
                      status=$(docker inspect --format='{{.State.Health.Status}}' mysql || echo "none")
                      if [ "$status" = "healthy" ]; then
                        echo "✅ MySQL is healthy!"
                        break
                      fi
                      echo "MySQL not ready yet... waiting 10s"
                      sleep 10
                    done
                    '''

                    // ✅ Check Flask health endpoint
                    sh '''
                    echo "🔍 Checking Flask health..."
                    for i in {1..10}; do
                      if curl -sf http://localhost:5000/health; then
                        echo "✅ Flask is healthy!"
                        exit 0
                      fi
                      echo "Flask not ready yet... waiting 10s"
                      sleep 10
                    done
                    echo "❌ Flask health check failed."
                    docker logs flask-app
                    exit 1
                    '''

                    sh "docker ps"
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ Build successful",
                body: "Good news! The build and deployment were successful.",
                to: 'souravpatel65@gmail.com'
            )
        }
        failure {
            emailext(
                subject: "❌ Build failed",
                body: "Unfortunately, the build or deployment failed. Please check Jenkins logs for details.",
                to: 'souravpatel65@gmail.com'
            )
        }
    }
}
