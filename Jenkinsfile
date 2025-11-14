pipeline {
    agent any

    environment {
        // Path to your kubeconfig on the Windows agent. Update if needed.
        KUBECONFIG = "C:/Users/dmand/.kube/config"

        // Docker image base name (no tag)
        DOCKER_IMAGE = "employeeprofilemanagement_image"

        // PostgreSQL database URL (change if needed)
        DB_URL = "jdbc:postgresql://host.docker.internal:5432/epms_db"
        DB_USERNAME = "postgres"
        DB_PASSWORD = "Kaeith@04"

        // Kubernetes Namespace (must match namespace.yaml)
        K8S_NAMESPACE = "employeemanagementsystem"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/KaeithEmmanuel/employee_profile_mangement'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🚀 Building Docker image (Windows agent)..."
                // Use Windows bat command. Jenkins provides BUILD_NUMBER environment variable.
                bat """
                    docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% -f Dockerfile .
                """
            }
        }

        stage('Run Container Locally (optional)') {
            steps {
                echo "🐳 Running container locally on the Windows agent (optional)"
                // stop + remove any existing container with same name, then run
                bat """
                    if exist "%ProgramFiles%\\docker\\docker.exe" (
                        echo Docker is installed
                    )
                    REM Stop & remove if existing
                    for /f "tokens=*" %%i in ('docker ps -aq -f name=employeeprofilemanagement') do (
                        docker stop %%i || echo ignore
                        docker rm -f %%i || echo ignore
                    ) || echo "No running/existing container to remove"

                    docker run -d --name employeeprofilemanagement ^
                        -e SPRING_DATASOURCE_URL="%DB_URL%" ^
                        -e SPRING_DATASOURCE_USERNAME="%DB_USERNAME%" ^
                        -e SPRING_DATASOURCE_PASSWORD="%DB_PASSWORD%" ^
                        -p 8200:8200 ^
                        %DOCKER_IMAGE%:%BUILD_NUMBER%
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "📦 Applying namespace and service, then updating deployment image (Windows)"
                // Apply namespace and service, then use kubectl set image to update the deployment image
                bat """
                    kubectl --kubeconfig="%KUBECONFIG%" apply -f namespace.yaml
                    kubectl --kubeconfig="%KUBECONFIG%" -n %K8S_NAMESPACE% apply -f service.yaml
                    kubectl --kubeconfig="%KUBECONFIG%" -n %K8S_NAMESPACE% apply -f deployment.yaml

                    REM Update the deployment image using kubectl set image (works cross-platform)
                    kubectl --kubeconfig="%KUBECONFIG%" -n %K8S_NAMESPACE% set image deployment/employeemanagementsystem-deployment \
                      employeemanagementsystem-container=%DOCKER_IMAGE%:%BUILD_NUMBER% --record

                    REM Wait for rollout to finish
                    kubectl --kubeconfig="%KUBECONFIG%" -n %K8S_NAMESPACE% rollout status deployment/employeemanagementsystem-deployment --timeout=120s
                """
            }
        }
    }

    post {
        success {
            echo "🎉 SUCCESS: Build and Kubernetes deployment completed!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
