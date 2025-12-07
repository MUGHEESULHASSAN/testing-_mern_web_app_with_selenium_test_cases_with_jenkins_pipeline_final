pipeline {
    agent { label 'ubuntu3' }

    environment {
        DOCKER_NETWORK = 'ci-network'
    }

    stages {
        stage('Clone Main Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Zillehuma626/ASSIGNMENT_3.git'
            }
        }

        stage('Build & Run Main Containers') {
            steps {
                sh '''
                sudo docker compose down || true
                sudo docker compose up -d --build
                '''
            }
        }

        stage('Verify Running Containers') {
            steps {
                sh 'sudo docker ps'
            }
        }

        stage('Run Selenium Tests') {
            steps {
                dir('selenium-tests') {
                    sh '''
                    # Wait until frontend is ready
                    while ! nc -z user-frontend-ci 5173; do
                        echo "Waiting for frontend..."
                        sleep 2
                    done

                    # Build Selenium test image
                    sudo docker build -t selenium-tests .

                    # Run Selenium container inside Docker network
                    sudo docker run --rm --network=${DOCKER_NETWORK} -e BASE_URL="http://user-frontend-ci:5173" selenium-tests
                    '''
                }
            }
        }

        stage('Show Logs') {
            steps {
                sh 'sudo docker compose logs --tail=50'
            }
        }
    }

    post {
        success {
            echo "✅ CI Build Completed Successfully!"
        }
        failure {
            echo "❌ Build Failed. Check logs."
        }
    }
}
