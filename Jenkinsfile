pipeline {
    agent any

    environment {
        // Uncomment below if pushing to DockerHub
        // DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = "employeeprofilemanagement_image"
        CONTAINER_NAME = "employeeprofilemanagement"

        // Database URL used when dockerization is done for the application
        DB_URL = "jdbc:postgresql://host.docker.internal:5432/epms_db"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/KaeithEmmanuel/employee_profile_mangement.git'
            }
        }

        stage('Build') {
            steps {
                // Build docker image using Dockerfile
                // BUILD_NUMBER is provided by Jenkins automatically
                bat "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -f Dockerfile ."
            }
        }

        stage('Cleanup existing container') {
            steps {
                script {
                    // This batch script will look for any container with the same name,
                    // stop and forcibly remove it if found.
                    bat """
@echo off
setlocal enabledelayedexpansion

echo Checking for existing container named %CONTAINER_NAME%...

rem get container id(s) that match the name filter (quiet mode - just IDs)
for /f "delims=" %%i in ('docker ps -a -q -f "name=%CONTAINER_NAME%"') do (
    set "CID=%%i"
)

if defined CID (
    echo Found existing container with ID: %CID%
    echo Stopping and removing container %CONTAINER_NAME%...
    docker rm -f %CONTAINER_NAME%
    if errorlevel 1 (
        echo WARNING: failed to remove container %CONTAINER_NAME% (docker rm returned non-zero)
        rem continue anyway; next run may fail if container still exists
    ) else (
        echo Successfully removed previous container %CONTAINER_NAME%.
    )
) else (
    echo No existing container named %CONTAINER_NAME% found. Continuing.
)

endlocal
"""
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    // Run new container (detached) with the desired env var and port mapping
                    bat """
echo Running new container %CONTAINER_NAME% from image ${DOCKER_IMAGE}:${BUILD_NUMBER}...
docker run -e DB_URL="%DB_URL%" -d --name %CONTAINER_NAME% -p 8200:8200 ${DOCKER_IMAGE}:${BUILD_NUMBER}
if errorlevel 1 (
    echo ERROR: docker run failed.
    exit /b 1
)
echo Container started.
"""
                }
            }
        }
    }

    post {
        success {
            echo "✅ Checkout, Build, Dockerize & Deploy completed successfully!"
        }
        failure {
            echo "❌ Build or deployment failed!"
        }
    }

}
