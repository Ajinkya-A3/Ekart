
# 🚀 Ekart CI/CD Pipeline with Jenkins, Docker, Argo CD, and GitOps

This project implements a complete DevSecOps CI/CD pipeline for a Java-based e-commerce application (**Ekart**) using Jenkins, SonarQube, Docker, Nexus, Trivy, and Argo CD with GitOps methodology.

---

## ⚙️ Environment Setup

### ☕ Install JDK 17 on Ubuntu

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
```

---

### 🔧 Install Jenkins on Ubuntu

```bash
# Step 1: Add Jenkins repository and key
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee   /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]   https://pkg.jenkins.io/debian binary/ | sudo tee   /etc/apt/sources.list.d/jenkins.list > /dev/null

# Step 2: Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Step 3: Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Step 4: Access Jenkins in browser
# URL: http://<your-server-ip>:8080
```

---

## 📌 Project Overview

This pipeline ensures:
- Code compilation and testing
- Code quality analysis (SonarQube)
- Secure build and packaging
- Docker image scanning (Trivy)
- Artifact upload to Nexus
- GitOps-style deployment to Kubernetes using Argo CD

---

## 🛠️ Tools & Technologies

| Tool           | Purpose                                     |
|----------------|---------------------------------------------|
| Jenkins        | CI/CD Orchestration                         |
| Maven          | Project Build Tool                          |
| JDK 17         | Java Compilation                            |
| SonarQube      | Static Code Analysis                        |
| Nexus          | Artifact Repository                         |
| Docker         | Containerization                            |
| Trivy          | Image Vulnerability Scanning                |
| Argo CD        | Continuous Deployment via GitOps            |
| GitHub         | Version Control & GitOps Manifest Repository|

---

## 🧱 Project Structure

```
Ekart/
├── docker/
│   └── Dockerfile                  # Docker build file
├── k8s/
│   └── deploymentservice.yml      # Kubernetes deployment manifest
├── src/                           # Java source code
├── pom.xml                        # Maven project file
├── Jenkinsfile                    # CI/CD pipeline definition
└── trivy-report.txt               # Output of Trivy scan
```

---

## 🔄 CI/CD Workflow (Jenkins Pipeline)

### Stages:
1. **Git Checkout**  
   Clones the `main` branch from GitHub.

2. **Compile & Unit Test**  
   Compiles the Java code and optionally runs tests.

3. **SonarQube Scan**  
   Analyzes the code using SonarQube for bugs and code smells.

4. **Build & Package**  
   Packages the application using Maven.

5. **Deploy to Nexus**  
   Uploads `.jar`/`.war` to Nexus Repository.

6. **Docker Image Build & Trivy Scan**  
   Builds a Docker image and scans it using Trivy for vulnerabilities.

7. **Push Docker Image**  
   Pushes the tagged image to Docker Hub.

8. **GitOps Deployment**  
   - Updates the image tag in `k8s/deploymentservice.yml`
   - Commits the change to GitHub

9. **Argo CD Sync**  
   Triggers Argo CD sync to deploy the updated image in Kubernetes.

---

## 🚀 Argo CD Setup in Kubernetes

```bash
# Step 1: Create Argo CD namespace and install
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Step 2: Expose Argo CD (via NodePort)
kubectl patch svc argocd-server -n argocd -p '{
  "spec": {
    "type": "NodePort",
    "ports": [{"port": 80, "targetPort": 8080, "nodePort": 32000}]
  }
}'

# Step 3: Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret   -o jsonpath="{.data.password}" | base64 -d && echo

# Step 4: Access Argo CD in browser
# URL: http://<your-vm-ip>:32000
# Username: admin
# Password: (output from above)

# Step 5: Install Argo CD CLI
VERSION=$(curl -s https://api.github.com/repos/argoproj/argo-cd/releases/latest   | grep tag_name | cut -d '"' -f 4)

curl -sSL -o argocd   "https://github.com/argoproj/argo-cd/releases/download/${VERSION}/argocd-linux-amd64"

chmod +x argocd
sudo mv argocd /usr/local/bin/
```

---

## 🔐 Credentials Used

| Credential ID     | Usage                          |
|-------------------|--------------------------------|
| `docker-cred`     | For DockerHub login            |
| `git-credentials` | For pushing manifest to GitHub |
| `argocd-cred`     | For Argo CD login              |

---

## 🌐 Accessing Argo CD

- URL: `http://34.47.186.224:32000`
- App Name: `ekart`

---

## 📈 Future Improvements

- Enable OWASP Dependency-Check (commented in Jenkinsfile)
- Archive Trivy and Sonar reports in Jenkins
- Add integration and end-to-end test stages

---

## ✍️ Author

**Ajinkya**  
Cloud & DevOps Enthusiast | CI/CD | Kubernetes | GitOps  
[GitHub Profile](https://github.com/Ajinkya-A3)

---

## 📄 License

This project is licensed under the MIT License.
