pipeline {
    agent { label 'ubuntu3' }

    environment {
        // Define standard variables
        DOCKER_COMPOSE_CMD = 'docker-compose'
        REPO_URL = 'https://github.com/MUGHEESULHASSAN/Deploying_3_Tier_Web_App_Using_Docker_Compose_And_Jenkins_Pipeline.git'
        BRANCH = 'main'
    }

    stages {
        // --- STAGE 1: Checkout Code ---
        stage('Checkout Code') {
            steps {
                echo "Cloning repository: ${env.REPO_URL}"
                git branch: env.BRANCH, url: env.REPO_URL
            }
        }

        // --- STAGE 2: Local Frontend Build ---
        // This stage builds the static files that are mounted by docker-compose.yml
        stage('Local Frontend Build') {
            steps {
                echo 'Building static frontends on Jenkins agent...'
                script {
                    // Build User Frontend
                    sh 'cd user-frontend && npm install && npm run build'
                    
                    // Build Admin Frontend
                    sh 'cd admin-frontend && npm install && npm run build'
                }
                echo '✅ Frontend static files built into /dist directories.'
            }
        }

        // --- STAGE 3: Deploy Application ---
        stage('Deploy Application') {
            steps {
                script {
                    echo 'Stopping old containers...'
                    // Down command must use the --no-env-file flag to avoid errors if the .env file is missing
                    sh "${env.DOCKER_COMPOSE_CMD} down --no-env-file || true" 

                    echo 'Running new containers...'
                    
                    // Use bash -c to ensure the command executes correctly and avoids "help text" failure.
                    // Compose will read variables from ./backend/.env file.
                    sh "bash -c '${env.DOCKER_COMPOSE_CMD} up -d --pull always'"
                }
            }
        }

        // --- STAGE 4: Verify Deployment ---
        stage('Verify Deployment') {
            steps {
                echo 'Checking running containers...'
                sh 'docker ps'
            }
        }
    }

    // --- POST: Cleanup ---
    post {
        always {
            echo 'Cleaning up containers after build...'
            // Use --no-env-file to prevent the cleanup from crashing due to missing secrets/env variables
            sh "${env.DOCKER_COMPOSE_CMD} down --no-env-file || true" 
        }
    }
}