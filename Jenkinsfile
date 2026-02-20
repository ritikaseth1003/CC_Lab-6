pipeline {
    agent any
    
    stages {
        stage('Build Backend Image') {
            steps {
                sh 'docker build -t backend-app backend/'
            }
        }
        
        stage('Create Network') {
            steps {
                sh '''
                    docker network rm app-network || true
                    docker network create app-network
                '''
            }
        }
        
        stage('Deploy Backends') {
            steps {
                sh '''
                    docker rm -f backend1 backend2 || true
                    docker run -d --name backend1 --network app-network -p 8081:8080 backend-app
                    docker run -d --name backend2 --network app-network -p 8082:8080 backend-app
                    echo "Waiting for backends to initialize..."
                    sleep 10
                '''
            }
        }
        
        stage('Start NGINX') {
            steps {
                sh '''
                    docker rm -f nginx-lb || true
                    docker run -d --name nginx-lb --network app-network -p 80:80 nginx
                    echo "Waiting for NGINX to start..."
                    sleep 5
                    
                    # Update nginx config with correct container names
                    cat > nginx/default.conf << 'INNER_EOF'
upstream backend_servers {
    server backend1:8080;
    server backend2:8080;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://backend_servers;
    }
}
INNER_EOF
                    
                    docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    docker exec nginx-lb nginx -t && docker exec nginx-lb nginx -s reload
                    echo "NGINX configured successfully!"
                '''
            }
        }
        
        stage('Verify') {
            steps {
                sh '''
                    echo "===== Running containers ====="
                    docker ps | grep -E "backend|nginx"
                    
                    echo "===== Testing backends directly ====="
                    curl -s http://localhost:8081 || echo "Backend1 not ready"
                    curl -s http://localhost:8082 || echo "Backend2 not ready"
                    
                    echo "===== Testing load balancer ====="
                    curl -s http://localhost || echo "Load balancer not ready"
                    
                    echo "===== Network inspection ====="
                    docker network inspect app-network | grep -A 10 "Containers"
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
