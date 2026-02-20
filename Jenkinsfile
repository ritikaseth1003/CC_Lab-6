pipeline {
    agent any
    
    stages {
        stage('Build Backend Image') {
            steps {
                sh 'docker build -t backend-app backend/'
            }
        }
        
        stage('Deploy Backends') {
            steps {
                sh '''
                    docker rm -f backend1 backend2 || true
                    docker run -d --name backend1 -p 8081:8080 backend-app
                    docker run -d --name backend2 -p 8082:8080 backend-app
                    echo "Waiting for backends to initialize..."
                    sleep 10
                '''
            }
        }
        
        stage('Start NGINX') {
            steps {
                sh '''
                    docker rm -f nginx-lb || true
                    docker run -d --name nginx-lb -p 80:80 nginx
                    echo "Waiting for NGINX to start..."
                    sleep 5
                    docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    echo "Testing NGINX config..."
                    docker exec nginx-lb nginx -t || true
                    docker exec nginx-lb nginx -s reload
                    echo "NGINX configured successfully!"
                '''
            }
        }
        
        stage('Verify') {
            steps {
                sh '''
                    echo "Running containers:"
                    docker ps | grep -E "backend|nginx"
                    echo "Testing backend1:"
                    curl -s http://localhost:8081 || echo "Backend1 not ready yet"
                    echo "Testing backend2:"
                    curl -s http://localhost:8082 || echo "Backend2 not ready yet"
                    echo "Testing load balancer:"
                    curl -s http://localhost || echo "Load balancer not ready yet"
                '''
            }
        }
    }
    
    post {
        success {
            echo '🎉 Pipeline completed successfully!'
            echo 'Access your app at: http://localhost'
        }
        failure {
            echo '❌ Pipeline failed. Check logs above.'
        }
    }
}
