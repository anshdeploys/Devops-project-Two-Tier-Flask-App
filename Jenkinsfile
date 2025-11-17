pipeline{
    
    agent any;
    
    stages{
        stage("Code"){
            steps{
                git url:"https://github.com/anshdeploys/Devops-project-Two-Tier-Flask-App.git", branch:"main"
                echo "Code pulled successfully"
            }
        }
        stage("Build"){
            steps{
                sh "docker build -t Two-Tier-Flask-App:latest ."
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
                    sh "docker image tag Two-Tier-Flask-App ${env.DockerUser}/Two-Tier-Flask-App:latest"
                    sh "docker push ${env.DockerUser}/Two-Tier-Flask-App:latest"
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
                sh "docker compose up -d --build"
                echo "Docker compose done"
            }
        }
    }
}
