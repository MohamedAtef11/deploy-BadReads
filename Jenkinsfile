pipeline {
    agent any

    stages {
        stage('Build local') {
            steps {
                sh "sudo docker build -t build-frontend:1.0 ./client"
                sh "sudo docker build -t build-backend:1.0 ./server"
                sh "sudo docker build -t build-nginx:1.0 ./nginx"
            }
        }
        stage('Test') {
            steps {
                sh "sudo docker run build-frontend:1.0 npm run test"
                sh "sudo docker run build-backend:1.0 npm run test"
            }
        }
        stage('Build and push images ') {
            steps {
                sh "sudo docker login -u '$Dockerhub_username' -p '$Dockerhub_password'"

                sh "sudo docker build -t muhammadatef/BadReads-nginx:latest -t muhammadatef/BadReads-nginx:\$(git rev-parse HEAD) -f ./nginx/Dockerfile ./nginx"
                sh "sudo docker push muhammadatef/BadReads-nginx:latest" 
                sh "sudo docker push muhammadatef/BadReads-nginx:\$(git rev-parse HEAD)" 

                sh "sudo docker build -t muhammadatef/BadReads-frontend:latest -t muhammadatef/BadReads-frontend:\$(git rev-parse HEAD) -f ./badreads-frontend/Dockerfile ./badreads-frontend"
                sh "sudo docker push muhammadatef/BadReads-frontend:latest" 
                sh "sudo docker push muhammadatef/BadReads-frontend:\$(git rev-parse HEAD)" 


                sh "sudo docker build -t muhammadatef/BadReads-backend:latest -t muhammadatef/BadReads-backend:\$(git rev-parse HEAD) -f ./badreads-backend/Dockerfile ./badreads-backend"
                sh "sudo docker push muhammadatef/BadReads-backend:latest" 
                sh "sudo docker push muhammadatef/BadReads-backend:\$(git rev-parse HEAD)" 
                
            }
        }

         stage('copy files to AWS server') {
            
            steps {
                sh "sudo ssh -i /home/mo-atef/Downloads/aws-key.pem ec2-user@18.222.192.38 '[ ! -d '/home/ec2-user/pro' ] && mkdir /home/ec2-user/pro;echo Directory-exist'"
                sh "sudo rsync -rv --update -e 'ssh -i /home/mo-atef/Downloads/aws-key.pem' -r /var/lib/jenkins/workspace/test1/docker-compose.yml ec2-user@18.222.192.38:/home/ec2-user/pro"
                sh "sudo rsync -rv --update -e 'ssh -i /home/mo-atef/Downloads/aws-key.pem' -r  /var/lib/jenkins/workspace/test1/badreads-frontend/public/ ec2-user@18.222.192.38:/home/ec2-user/pro/web/" 
            }
        }
        
        stage('Deploy on AWS server') {
            
            steps {
                sh "sudo ssh -tt -i /home/mo-atef/Downloads/aws-key.pem ec2-user@18.222.192.38 'sudo docker-compose -f /home/ec2-user/pro/docker-compose.yml  down'"
                sh "sudo ssh -tt -i /home/mo-atef/Downloads/aws-key.pem ec2-user@18.222.192.38 'sudo docker-compose -f /home/ec2-user/pro/docker-compose.yml  up --build '"
            }
        }
    }
}