
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Vrushk13/Nexvion.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t nexvion-website:1.0 .'
            }
        }

        stage('Deploy Website') {
            steps {
                sh '''
                    if docker container inspect nexvion-website-container >/dev/null 2>&1; then
                        docker rm -f nexvion-website-container
                    fi

                    docker run -d \
                        --name nexvion-website-container \
                        --restart unless-stopped \
                        -p 8082:80 \
                        nexvion-website:1.0
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    for i in $(seq 1 10); do
                        if curl -fsS http://localhost:8082/ -o /dev/null; then
                            echo "Nexvion website is healthy!"
                            exit 0
                        fi
                        sleep 3
                    done

                    echo "Nexvion website health check failed"
                    docker logs nexvion-website-container
                    exit 1
                '''
            }
        }
    }
}
