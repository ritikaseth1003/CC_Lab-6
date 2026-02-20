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
                    sleep 5
                '''
            }
        }
        
        stage('Start NGINX') {
            steps {
                sh '''
                    docker rm -f nginx-lb || true
                    docker run -d --name nginx-lb -p 80:80 nginx
                    sleep 3
                    docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
