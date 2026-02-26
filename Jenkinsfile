pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
                echo '✅ Backend image built successfully.'
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network create app-network || true
                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # Give Docker time to register containers in network DNS
                sleep 5
                '''
                echo '✅ Backend containers deployed successfully.'
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true

                docker run -d --name nginx-lb \
                --network app-network \
                -p 80:80 nginx

                # Wait for nginx to initialize
                sleep 5

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Wait for backend DNS resolution
                sleep 5

                docker exec nginx-lb nginx -s reload
                '''
                echo '✅ NGINX load balancer deployed successfully.'
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline executed successfully! All services are up and running.'
        }
        failure {
            echo '❌ Pipeline failed. Check console logs for errors.'
        }
    }
}
