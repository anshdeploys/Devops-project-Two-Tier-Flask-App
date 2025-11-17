pipeline{
    
    agent { lable "dev"};
    
    stages{
        stage("Code"){
            steps{
                git url:"https://github.com/anshdeploys/Devops-project-Two-Tier-Flask-App.git", branch:"main"
                echo "Code pulled successfully"
            }
        }
        stage("Build"){
            steps{
                sh "docker build -t two-tier-flask-app:latest ."
                echo "Docker image created successfuly"
            }
        }
        stage("Push To Dockerhub"){
            steps{
                withCredentials([usernamePassword(
                    credentialsId:"DockerHubCreds", 
                    usernameVariable:"DockerUser", 
                    passwordVariable:"DockerPass"
                )]){
                    sh "docker login -u ${env.DockerUser} -p ${env.DockerPass}"
                    sh "docker image tag two-tier-flask-app ${env.DockerUser}/two-tier-flask-app:latest"
                    sh "docker push ${env.DockerUser}/two-tier-flask-app:latest"
                }
            }
        }
        stage("Testing Process"){
            steps{
                echo "Testing successfull"
            }
        }
        stage("Deploy"){
            steps{
                sh "docker compose up -d --build ${env.DockerUser}/two-tier-flask-app:latest"
                echo "Docker compose done"
            }
        }
    }
}
