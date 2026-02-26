pipeline {
    agent any
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network create lab6-network || true
                docker rm -f backend1 backend2 || true
                docker run -d --name backend1 --network lab6-network backend-app
                docker run -d --name backend2 --network lab6-network backend-app
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                docker run -d --name nginx-lb -p 80:80 --network lab6-network nginx
                sleep 2
                if [ -f nginx/nginx.conf ]; then
                    docker cp nginx/nginx.conf nginx-lb:/etc/nginx/conf.d/default.conf
                elif [ -f nginx/default.conf ]; then
                    docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                fi
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
