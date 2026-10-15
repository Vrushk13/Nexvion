
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
                sh 'docker build --pull -t nexvion-website:1.2 .'
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy:latest image \
                        --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        nexvion-website:1.2
                '''
            }
        }

        stage('Deploy Website') {
            steps {
                sh '''
                    # Remove any leftover temporary container
                    docker rm -f nexvion-website-container-new 2>/dev/null || true

                    # Start the new image on temporary port 8083
                    docker run -d \
                        --name nexvion-website-container-new \
                        -p 8083:80 \
                        nexvion-website:1.2

                    # Test the new website before replacing the old one
                    TEST_OK=0

                    for i in $(seq 1 10); do
                        if curl -fsS http://localhost:8083/ -o /dev/null; then
                            TEST_OK=1
                            break
                        fi
                        sleep 3
                    done

                    if [ "$TEST_OK" -ne 1 ]; then
                        echo "New website failed its test."
                        docker logs nexvion-website-container-new
                        docker rm -f nexvion-website-container-new
                        exit 1
                    fi

                    # Remove the temporary test container
                    docker rm -f nexvion-website-container-new

                    # Replace the old website on port 8082
                    docker rm -f nexvion-website-container 2>/dev/null || true

                    docker run -d \
                        --name nexvion-website-container \
                        --restart unless-stopped \
                        -p 8082:80 \
                        nexvion-website:1.2

                    echo "Deployment completed."
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

                    echo "Nexvion website health check failed."
                    docker logs nexvion-website-container
                    exit 1
                '''
            }
        }
    }
}
