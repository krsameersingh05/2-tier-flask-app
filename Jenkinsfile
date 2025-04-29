pipeline{
    agent any
     
    stages{
        stage("Clone the Code From GitHub"){
            steps{
                echo "Pull the Code from GitHub"
                git url: "https://github.com/ahmednisarhere/2-tier-flask-app.git", branch: "main"
            }
        }
        
        stage("Build & Test the Code"){
            steps{
                 // Retrieve password from Jenkins credentials
                    echo"Build the Code"
                    sh"docker-compose down && docker-compose --env-file ~/.env up -d"
                    sh"docker cp message.sql \$(docker ps -f name=mysql -q):/docker-entrypoint-initdb.d/"
                    echo"Test the Code"
                    sleep time: 35, unit:'SECONDS'
                    sh'curl -Is http://localhost:5000 | head -n 1'
                    
            }
                    
                
        }
        stage("Push the Image to DockerHub"){
            steps{
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"pass",usernameVariable:"user")]){
                    sh "docker login -u ${env.user} -p ${env.pass}"
                    sh "docker commit \$(docker ps -f name=mysql -q) ${env.user}/two-tier-db:latest"
                    sh "docker commit \$(docker ps -f name=backend -q) ${env.user}/two-tier-app:latest"
                    sh "docker push ${env.user}/two-tier-app:latest"
                    sh "docker push ${env.user}/two-tier-db:latest"
                    sh "docker-compose down"
                    echo"Image Pushed"
                }
            }
        }
            
        
        stage("Deploy to AWS Server"){
                steps{
                    withCredentials([sshUserPrivateKey(credentialsId:"my-key", keyFileVariable:"MY_SSH_KEY1", usernameVariable:"SSH_USER1")]){
                        sh "ssh -o StrictHostKeyChecking=no -i ${MY_SSH_KEY1} ${SSH_USER1}  'sudo rm -rf 2-tier-flask-app && git clone -b prod https://github.com/ahmednisarhere/2-tier-flask-app.git && pwd && cd 2-tier-flask-app && docker-compose up -d'"
                        
                    }
        
                    echo"Deployed"
                }
            }
        }
    }   
