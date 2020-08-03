pipeline {
    agent any

    stages {
        stage('Build local') {
            steps {
                sh " docker build -t build-frontend:1.0 ./client"
                sh " docker build -t build-backend:1.0 ./server"
                sh " docker build -t build-nginx:1.0 ./nginx"
            }
        }
        stage('Test') {
            steps {
                sh " docker run build-frontend:1.0 npm run test"
                sh " docker run build-backend:1.0 npm run test"
            }
        }
        stage('Build and push images ') {
            steps {
                sh " docker login -u '$Dockerhub_username' -p '$Dockerhub_password'"

                sh " docker build -t muhammadatef/BadReads-nginx:latest -t muhammadatef/BadReads-nginx:\$(git rev-parse HEAD) -f ./nginx/Dockerfile ./nginx"
                sh " docker push muhammadatef/BadReads-nginx:latest" 
                sh " docker push muhammadatef/BadReads-nginx:\$(git rev-parse HEAD)" 

                sh " docker build -t muhammadatef/BadReads-frontend:latest -t muhammadatef/BadReads-frontend:\$(git rev-parse HEAD) -f ./badreads-frontend/Dockerfile ./badreads-frontend"
                sh " docker push muhammadatef/BadReads-frontend:latest" 
                sh " docker push muhammadatef/BadReads-frontend:\$(git rev-parse HEAD)" 


                sh " docker build -t muhammadatef/BadReads-backend:latest -t muhammadatef/BadReads-backend:\$(git rev-parse HEAD) -f ./badreads-backend/Dockerfile ./badreads-backend"
                sh " docker push muhammadatef/BadReads-backend:latest" 
                sh " docker push muhammadatef/BadReads-backend:\$(git rev-parse HEAD)" 
                
            }
        }

         stage('copy files to AWS server') {
            
            steps {
                sh " ssh -i /home/mo-atef/Downloads/aws-key.pem ec2-user@18.222.192.38 '[ ! -d '/home/ec2-user/pro' ] && mkdir /home/ec2-user/pro;echo Directory-exist'"
                sh " rsync -rv --update -e 'ssh -i /home/mo-atef/Downloads/aws-key.pem' -r /var/lib/jenkins/workspace/test1/docker-compose.yml ec2-user@18.222.192.38:/home/ec2-user/pro"
                sh " rsync -rv --update -e 'ssh -i /home/mo-atef/Downloads/aws-key.pem' -r  /var/lib/jenkins/workspace/test1/badreads-frontend/public/ ec2-user@18.222.192.38:/home/ec2-user/pro/web/" 
            }
        }
        
        stage('Deploy on AWS server') {
            
            steps {
                sh " ssh -tt -i /home/mo-atef/Downloads/aws-key.pem ec2-user@18.222.192.38 ' docker-compose -f /home/ec2-user/pro/docker-compose.yml  down'"
                sh " ssh -tt -i /home/mo-atef/Downloads/aws-key.pem ec2-user@18.222.192.38 ' docker-compose -f /home/ec2-user/pro/docker-compose.yml  up --build '"
            }
        }
    }
}