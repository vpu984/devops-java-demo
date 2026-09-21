DevOps Java Demo – Complete Setup & CI/CD Reference

GitHub → Jenkins → Maven → Docker → Docker Hub → Kubernetes → Webhook

A reusable step-by-step record of the working Windows + WSL Ubuntu project, including Jenkins CI/CD, Docker Hub, Kubernetes deployment, application verification, and GitHub webhook triggering.

1. Project Architecture
GitHub
   ↓
GitHub Webhook
   ↓
ngrok
   ↓
Jenkins :8080
   ↓
Git Checkout
   ↓
Maven Build
   ↓
Maven Test
   ↓
Docker Build
   ↓
Docker Hub Push
   ↓
Kubernetes (Docker Desktop)
   ↓
2 Spring Boot Pods
   ↓
Kubernetes Service
   ↓
Application Verification

3. Project Details
 
•	GitHub user: vpu984
•	GitHub repository: https://github.com/vpu984/devops-java-demo.git
•	Docker Hub user: vishal984
•	Docker image: vishal984/devops-java-demo:1.0
•	Jenkins: http://localhost:8080
•	Spring Boot port: 8081
•	Temporary port-forward: localhost:8082
•	Kubernetes context: docker-desktop
•	Kubernetes node: desktop-control-plane
•	Jenkins service account: LocalSystem

5. Environment and Files
   
•	Jenkins Home: C:\ProgramData\Jenkins\.jenkins
•	Jenkins uses Java 21 and runs on port 8080.
•	Docker Desktop provides Docker and Kubernetes.
•	Jenkins uses bat commands on Windows.
•	Jenkins Kubernetes kubeconfig: C:\ProgramData\Jenkins\.kube\config

7. Spring Boot and Docker Configuration
   
application.properties
server.port=8081
Dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]

9. Kubernetes deployment.yaml
    
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-java-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-java-demo
  template:
    metadata:
      labels:
        app: devops-java-demo
    spec:
      containers:
        - name: devops-java-demo
          image: vishal984/devops-java-demo:1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8081
---
apiVersion: v1
kind: Service
metadata:
  name: devops-java-demo-service
spec:
  type: NodePort
  selector:
    app: devops-java-demo
  ports:
    - port: 8081
      targetPort: 8081
      nodePort: 30081
      
6. Git Setup and Push
   
cd ~/devops-java-demo
git status
git remote -v
git branch
git add .
git commit -m "Initial DevOps project"
git push origin main
If GitHub asks for credentials, use the GitHub username and PAT as the password. Never paste a PAT into chat.

8. GitHub Personal Access Token
   
1.	GitHub → Profile → Settings → Developer settings → Personal access tokens.
2.	Use Fine-grained tokens → Generate new token.
3.	Select repository vpu984/devops-java-demo.
4.	For pushing repository files, grant Contents: Read and write.
5.	GitHub does not display the token value again after creation; save it securely.
   
8. Jenkins and Docker Verification

http://localhost:8080

docker version
docker login -u vishal984
docker build -t vishal984/devops-java-demo:1.0 .
docker push vishal984/devops-java-demo:1.0
The working Jenkins pipeline does not contain a Docker Login stage. Docker Push uses the authentication available to the Jenkins LocalSystem environment.
10. Kubernetes Configuration for Jenkins
kubectl config get-contexts
kubectl config use-context docker-desktop
kubectl get nodes
kubectl get pods -A

# WSL
cp ~/.kube/config /mnt/c/Users/vpuja/kubeconfig

# Copy to:
C:\ProgramData\Jenkins\.kube\config

# Grant SYSTEM access:
icacls C:\ProgramData\Jenkins\.kube\config /grant "NT AUTHORITY\SYSTEM":F
10. Final Jenkinsfile
pipeline {

    agent any

    environment {
        IMAGE_NAME = 'vishal984/devops-java-demo'
        IMAGE_TAG  = '1.0'
        KUBECONFIG = 'C:\\ProgramData\\Jenkins\\.kube\\config'
    }

    stages {

        stage('1. Git Check') {
            steps {
                echo 'STEP 1: Checking Git installation and repository...'
                bat 'git --version'
                bat 'git ls-remote https://github.com/vpu984/devops-java-demo.git'
                echo 'GitHub repository is accessible.'
            }
        }

        stage('2. Checkout') {
            steps {
                echo 'STEP 2: Checking out code from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/vpu984/devops-java-demo.git'
                echo 'GitHub checkout completed successfully.'
                bat 'git branch --show-current'
                bat 'git log -1 --oneline'
                bat 'git remote -v'
            }
        }

        stage('3. Maven Build') {
            steps {
                echo 'STEP 3: Building Spring Boot application...'
                bat 'mvn clean package -DskipTests'
                echo 'Maven build completed successfully.'
            }
        }

        stage('4. Maven Test') {
            steps {
                echo 'STEP 4: Running unit tests...'
                bat 'mvn test'
                echo 'Maven tests completed successfully.'
            }
        }

        stage('5. Docker Build') {
            steps {
                echo 'STEP 5: Building Docker image...'
                bat 'docker --version'
                bat 'docker build -t %IMAGE_NAME%:%IMAGE_TAG% .'
                echo 'Docker image built successfully.'
            }
        }

        stage('6. Docker Push') {
            steps {
                echo 'STEP 6: Pushing Docker image to Docker Hub...'
                bat 'docker push %IMAGE_NAME%:%IMAGE_TAG%'
                echo 'Docker image pushed successfully.'
            }
        }

        stage('7. Kubernetes Check') {
            steps {
                echo 'STEP 7: Checking Kubernetes cluster...'
                bat 'kubectl version --client'
                bat 'kubectl config current-context'
                bat 'kubectl get nodes'
                echo 'Kubernetes cluster is accessible.'
            }
        }

        stage('8. Kubernetes Deploy') {
            steps {
                echo 'STEP 8: Deploying application to Kubernetes...'
                bat 'kubectl apply -f deployment.yaml'
                echo 'Kubernetes deployment completed.'
            }
        }

        stage('9. Kubernetes Rollout') {
            steps {
                echo 'STEP 9: Waiting for Kubernetes rollout...'
                bat 'kubectl rollout status deployment/devops-java-demo --timeout=120s'
                echo 'Kubernetes rollout completed successfully.'
            }
        }

        stage('10. Kubernetes Verify') {
            steps {
                echo 'STEP 10: Verifying Kubernetes resources...'
                bat 'kubectl get deployments'
                bat 'kubectl get pods'
                bat 'kubectl get services'
                echo 'Kubernetes resources verified successfully.'
            }
        }

        stage('11. Application Verification') {
            steps {
                echo 'STEP 11: Starting Kubernetes port-forward...'

                powershell '''
                    $ErrorActionPreference = "Stop"

                    $process = Start-Process `
                        -FilePath "kubectl.exe" `
                        -ArgumentList "port-forward svc/devops-java-demo-service 8082:8081" `
                        -PassThru `
                        -WindowStyle Hidden

                    Set-Content -Path "portforward.pid" -Value $process.Id
                    Write-Host "Port-forward process started. PID:" $process.Id
                    Start-Sleep -Seconds 8
                '''

                echo 'Testing Spring Boot application...'
                bat 'curl.exe --fail --silent --show-error http://localhost:8082'
                echo 'Spring Boot application verification successful.'

                powershell '''
                    if (Test-Path "portforward.pid") {
                        $portForwardPid = Get-Content "portforward.pid"
                        Write-Host "Stopping port-forward PID:" $portForwardPid
                        Stop-Process -Id ([int]$portForwardPid) -Force -ErrorAction SilentlyContinue
                        Remove-Item "portforward.pid" -Force -ErrorAction SilentlyContinue
                        Write-Host "Port-forward stopped successfully."
                    }
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY!'
        }
        failure {
            echo 'CI/CD PIPELINE FAILED. Check the failed stage.'
        }
    }
}
11. Put Jenkinsfile in Git
cd ~/devops-java-demo
nano Jenkinsfile
# Paste Jenkinsfile, then Ctrl+O, Enter, Ctrl+X

git status
git add Jenkinsfile
git commit -m "Add complete Jenkins CI/CD pipeline"
git push origin main
12. Configure Jenkins to Read Jenkinsfile from Git
6.	Jenkins → devops-java-demo → Configure.
7.	Pipeline → Definition: Pipeline script from SCM.
8.	SCM: Git.
9.	Repository URL: https://github.com/vpu984/devops-java-demo.git
10.	Branch: */main.
11.	Script Path: Jenkinsfile.
12.	Save, then Build Now.
13. Application Verification
kubectl get deployments
kubectl get pods
kubectl get svc

kubectl port-forward svc/devops-java-demo-service 8082:8081

# In another terminal:
curl http://localhost:8082
Expected response:
Hello from Spring Boot!
14. GitHub Webhook with ngrok
Because Jenkins is local on localhost:8080, GitHub cannot directly reach it. ngrok provides a temporary public HTTPS tunnel.
ngrok http 8080
Working ngrok forwarding URL used in this project:
https://vilma-fluffiest-noniconoclastically.ngrok-free.dev
Do not use https://app.ngrok.ai; that is the ngrok website. Keep the ngrok terminal running during webhook testing.
15. Jenkins Webhook Configuration
13.	Jenkins → Manage Jenkins → System → Jenkins Location.
14.	Set Jenkins URL to the current ngrok HTTPS URL, e.g. https://vilma-fluffiest-noniconoclastically.ngrok-free.dev/
15.	Jenkins → devops-java-demo → Configure → Build Triggers.
16.	Enable: GitHub hook trigger for GITScm polling.
17.	Save.
16. GitHub Webhook Configuration
18.	GitHub → vpu984/devops-java-demo → Settings → Webhooks → Add webhook.
19.	Payload URL: https://vilma-fluffiest-noniconoclastically.ngrok-free.dev/github-webhook/
20.	Content type: application/json.
21.	Select: Just the push event.
22.	Keep Active checked.
23.	Click Add webhook.
17. Test Webhook
cd ~/devops-java-demo
nano Jenkinsfile
# Make a small harmless change, save, and exit

git status
git add Jenkinsfile
git commit -m "Test GitHub webhook"
git push origin main
Expected flow:
git push
   ↓
GitHub push event
   ↓
GitHub Webhook
   ↓
ngrok
   ↓
Jenkins /github-webhook/
   ↓
Automatic Jenkins build
Verify GitHub → Settings → Webhooks → webhook → Recent Deliveries. A successful delivery should return HTTP 200. Then check Jenkins for the automatically created build.
18. Troubleshooting
Docker login
The working pipeline removed the Docker Login stage. Docker Push succeeded using the Docker authentication available to the Jenkins LocalSystem environment.
Kubernetes connection
The working configuration uses the docker-desktop context and C:\ProgramData\Jenkins\.kube\config.
Port-forward
The earlier start /B + timeout approach failed with Windows Jenkins redirection. The final pipeline uses PowerShell Start-Process, Start-Sleep, curl.exe, and a saved process ID.
Webhook / Reverse Proxy Monitor 403
A ReverseProxySetupMonitor 403 is not the webhook endpoint itself. Configure Jenkins Location with the current ngrok URL and use /github-webhook/ for the GitHub webhook.
19. Interview Explanation
“A developer pushes code to GitHub. A GitHub webhook triggers Jenkins. Jenkins checks out the main branch, builds the Spring Boot application with Maven, runs unit tests, builds a Docker image, and pushes it to Docker Hub. Jenkins then connects to the Docker Desktop Kubernetes cluster, applies the deployment manifest, waits for rollout, verifies pods and service, and performs application-level verification through a temporary kubectl port-forward.”
20. Final Checklist
•	Jenkinsfile committed to GitHub.
•	Jenkins uses Pipeline script from SCM.
•	Script Path is Jenkinsfile.
•	Docker image push succeeds.
•	Kubernetes context is docker-desktop.
•	Jenkins can read the kubeconfig.
•	Deployment has 2 replicas.
•	Service is devops-java-demo-service.
•	Spring Boot uses port 8081.
•	Temporary verification uses localhost:8082.
•	ngrok is running during webhook testing.
•	GitHub hook trigger for GITScm polling is enabled.
•	GitHub webhook uses /github-webhook/ and application/json.
•	Webhook is configured for push events.
•	Git push automatically starts Jenkins.
End of Reference Guide
