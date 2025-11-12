pipeline {
    agent any

    environment {
        // DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // if needed later
        DOCKER_IMAGE = "employeeprofilemanagement_image"
        DB_URL = "jdbc:postgresql://host.docker.internal:5432/epms_db"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/KaeithEmmanuel/employee_profile_mangement'
            }
        }

        stage('Build') {
            steps {
                // Use %BUILD_NUMBER% and %DOCKER_IMAGE% (Windows batch expansion)
                bat """
                    echo 🚀 Building Docker image...
                    echo Docker CLI: && where docker || echo "docker not found in PATH"
                    docker --version

                    REM Build image with tag using build number
                    docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% -f Dockerfile .
                """
            }
        }

        stage('Run Container') {
            steps {
                // Stop & remove existing container(s) with the same name, then run a new one
                bat """
                    echo 🧹 Checking if container(s) named 'employeeprofilemanagement' exist...

                    REM Find all matching container IDs (stopped or running)
                    for /f "tokens=*" %%i in ('docker ps -aq -f "name=employeeprofilemanagement"') do (
                        echo Found container %%i - stopping and removing...
                        docker stop %%i || echo "stop failed or container not running"
                        docker rm %%i || echo "remove failed"
                    )

                    echo 🐳 Running new Docker container...
                    REM Pass DB_URL from environment to container. Map port 8200
                    docker run -e DB_URL="%DB_URL%" -d --name employeeprofilemanagement -p 8200:8200 %DOCKER_IMAGE%:%BUILD_NUMBER%
                """
            }
        }
    }

    post {
        success {
            echo "✅ Checkout, Build, Dockerize & Deploy completed successfully!"
        }
        failure {
            echo "❌ Build failed!"
        }
    }
}
