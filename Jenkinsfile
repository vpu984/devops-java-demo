pipeline {

    agent any

    environment {
        IMAGE_NAME = 'vishal984/devops-java-demo'
        IMAGE_TAG  = '1.0'
        KUBECONFIG = 'C:\\ProgramData\\Jenkins\\.kube\\config'
    }

    stages {

        // =========================================================
        // STEP 1 - CHECKING OUT CODE FROM GITHUB
        // =========================================================
        stage('1. Checkout') {
            steps {
                echo 'STEP 1: Checking out code from GitHub...'

                git branch: 'main',
                    url: 'https://github.com/vpu984/devops-java-demo.git'

                echo 'GitHub checkout completed.'
            }
        }


        // =========================================================
        // STEP 2 - MAVEN BUILD START
        // =========================================================
        stage('2. Maven Build') {
            steps {
                echo 'STEP 2: Building Spring Boot application...'

                bat 'mvn clean package -DskipTests'

                echo 'Maven build completed successfully.'
            }
        }


        // =========================================================
        // STEP 3 - MAVEN TEST
        // =========================================================
        stage('3. Maven Test') {
            steps {
                echo 'STEP 3: Running unit tests...'

                bat 'mvn test'

                echo 'Maven tests completed successfully.'
            }
        }


        // =========================================================
        // STEP 4 - DOCKER BUILD
        // =========================================================
        stage('4. Docker Build') {
            steps {
                echo 'STEP 4: Building Docker image...'

                bat 'docker build -t %IMAGE_NAME%:%IMAGE_TAG% .'

                echo 'Docker image built successfully.'
            }
        }


        // =========================================================
        // STEP 5 - DOCKER HUB PUSH
        // =========================================================
        stage('5. Docker Push') {
            steps {
                echo 'STEP 5: Pushing Docker image to Docker Hub...'

                bat 'docker push %IMAGE_NAME%:%IMAGE_TAG%'

                echo 'Docker image pushed successfully.'
            }
        }


        // =========================================================
        // STEP 6 - KUBERNETES CHECK
        // =========================================================
        stage('6. Kubernetes Check') {
            steps {
                echo 'STEP 6: Checking Kubernetes cluster...'

                bat 'kubectl config current-context'
                bat 'kubectl get nodes'

                echo 'Kubernetes cluster is accessible.'
            }
        }


        // =========================================================
        // STEP 7 - KUBERNETES DEPLOYMENT
        // =========================================================
        stage('7. Kubernetes Deploy') {
            steps {
                echo 'STEP 7: Deploying application to Kubernetes...'

                bat 'kubectl apply -f deployment.yaml'

                echo 'Kubernetes deployment completed.'
            }
        }


        // =========================================================
        // STEP 8 - KUBERNETES ROLLOUT
        // =========================================================
        stage('8. Kubernetes Rollout') {
            steps {
                echo 'STEP 8: Waiting for Kubernetes rollout...'

                bat 'kubectl rollout status deployment/devops-java-demo --timeout=120s'

                echo 'Kubernetes rollout completed successfully.'
            }
        }


        // =========================================================
        // STEP 9 - VERIFY KUBERNETES RESOURCES
        // =========================================================
        stage('9. Kubernetes Verify') {
            steps {
                echo 'STEP 9: Verifying Kubernetes resources...'

                bat 'kubectl get deployments'
                bat 'kubectl get pods'
                bat 'kubectl get services'

                echo 'Kubernetes resources verified successfully.'
            }
        }


        // =========================================================
        // STEP 10 - APPLICATION VERIFICATION
        // =========================================================
        stage('10. Application Verification') {
            steps {

                echo 'STEP 10: Starting Kubernetes port-forward...'

                powershell '''
                    $ErrorActionPreference = "Stop"

                    # Start kubectl port-forward
                    $process = Start-Process `
                        -FilePath "kubectl.exe" `
                        -ArgumentList "port-forward svc/devops-java-demo-service 8082:8081" `
                        -PassThru `
                        -WindowStyle Hidden

                    # Save process ID
                    Set-Content -Path "portforward.pid" -Value $process.Id

                    Write-Host "Port-forward process started. PID:" $process.Id

                    # Give Kubernetes time to establish port-forward
                    Start-Sleep -Seconds 8
                '''

                echo 'Testing Spring Boot application...'

                bat '''
                    curl.exe --fail --silent --show-error http://localhost:8082
                '''

                echo ''
                echo 'Spring Boot application verification successful.'

                echo 'Stopping port-forward...'

                powershell '''
                    if (Test-Path "portforward.pid") {

                        $portForwardPid = Get-Content "portforward.pid"

                        Write-Host "Stopping port-forward PID:" $portForwardPid

                        Stop-Process `
                            -Id ([int]$portForwardPid) `
                            -Force `
                            -ErrorAction SilentlyContinue

                        Remove-Item "portforward.pid" -Force -ErrorAction SilentlyContinue

                        Write-Host "Port-forward stopped successfully."
                    }
                '''
            }
        }
    }


    // =============================================================
    // POST ACTIONS
    // =============================================================
    post {

        success {
            echo '''
=========================================================
             CI/CD PIPELINE SUCCESS
=========================================================

GitHub
   |
   v
Jenkins Checkout
   |
   v
Maven Build
   |
   v
Maven Test
   |
   v
Docker Build
   |
   v
Docker Hub Push
   |
   v
Kubernetes Check
   |
   v
Kubernetes Deploy
   |
   v
Kubernetes Rollout
   |
   v
Kubernetes Verification
   |
   v
Port Forward
   |
   v
Application Test
   |
   v
PIPELINE SUCCESS

Docker Image:
vishal984/devops-java-demo:1.0

Kubernetes Service:
devops-java-demo-service

Spring Boot Port:
8081

Temporary Local Port:
8082
=========================================================
'''
        }

        failure {
            echo '''
=========================================================
             CI/CD PIPELINE FAILED
=========================================================
Check the failed stage in Jenkins Console Output.
=========================================================
'''
        }
    }
}
