# Spring Boot CI/CD Pipeline — Jenkins, Docker & Kubernetes

A fully automated CI/CD pipeline built from scratch for a Spring Boot application. Every `git push` triggers an end-to-end pipeline that builds, tests, packages, containerizes, and deploys the app to a local Kubernetes cluster — no manual steps required.

Built as a hands-on learning project to understand DevOps fundamentals by implementing each stage manually before automating it, entirely on local infrastructure with **zero cloud costs**.

---

## 🚀 What This Project Does

```
git push → GitHub Webhook → Jenkins Pipeline → Build → Test → Package
                                                    ↓
                        Kubernetes (Minikube) ← Docker Image
                                                    ↓
                                          Live, running application
```

On every push to `main`, the pipeline automatically:
1. Compiles the Spring Boot source with Maven
2. Runs the unit test suite and publishes results
3. Packages a runnable JAR artifact
4. Builds and tags a Docker image
5. Deploys the image to a Kubernetes cluster with a rolling update
6. Verifies the rollout succeeded before marking the build complete

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Application | Java 21, Spring Boot |
| Build Tool | Maven |
| CI/CD | Jenkins (Declarative Pipeline) |
| Containerization | Docker |
| Orchestration | Kubernetes (Minikube) |
| Source Control | Git, GitHub |
| Trigger Mechanism | GitHub Webhooks (via ngrok tunnel) |

---

## 📁 Project Structure

```
.
├── src/
│   ├── main/java/com/example/demo/
│   │   ├── DemoApplication.java
│   │   └── HelloController.java
│   └── test/java/com/example/demo/
│       └── DemoApplicationTests.java
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── Dockerfile
├── Jenkinsfile
├── pom.xml
└── README.md
```

---

## ⚙️ Pipeline Stages

The Jenkinsfile defines the following stages, each isolating a single responsibility:

| Stage | Purpose |
|---|---|
| **Verify Java** | Confirms the correct JDK is available before building |
| **Build** | Compiles source code with Maven (`mvn compile`) |
| **Test** | Runs unit tests and publishes JUnit results |
| **Package** | Produces a runnable JAR and archives it as a build artifact |
| **Docker Build** | Builds and tags a Docker image, versioned by Jenkins build number |
| **Deploy to Kubernetes** | Rolls out the new image to the cluster and waits for a healthy rollout |

---

## 🧰 Prerequisites

To run this pipeline yourself, you'll need:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Jenkins](https://www.jenkins.io/) (running in Docker — see setup below)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/) + `kubectl`
- [ngrok](https://ngrok.com/) (for exposing local Jenkins to GitHub webhooks) — optional, "Poll SCM" works without it
- A GitHub account

**No cloud account or paid infrastructure required.** Everything runs locally.

---

## 🏁 Getting Started

### 1. Run Jenkins in Docker
```bash
docker network create jenkins

docker run -d --name jenkins \
  --network jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```
Visit `http://localhost:8080` and complete setup using the initial admin password:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 2. Configure Jenkins Tools
- **Manage Jenkins → Tools** → add a JDK installation pointing to `/opt/java/openjdk`
- Add a Maven installation with "Install automatically" enabled

### 3. Enable Docker inside Jenkins
```bash
docker exec -u root -it jenkins bash
apt-get update && apt-get install -y docker.io
chmod 666 /var/run/docker.sock
exit
```

### 4. Start Minikube
```bash
minikube start --driver=docker
kubectl create namespace staging
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### 5. Connect Jenkins to Minikube
```bash
docker network connect minikube jenkins
```
Copy your kubeconfig and minikube certs into the Jenkins container, and update the cluster server address to minikube's container IP. (Full step-by-step walkthrough in the [blog series](#) linked below.)

### 6. Create the Jenkins Pipeline job
- New Item → Pipeline → Definition: **Pipeline script from SCM**
- Point it at this repository, branch `main`, script path `Jenkinsfile`

### 7. Wire up the GitHub webhook
```bash
ngrok http 8080
```
Add the forwarded URL + `/github-webhook/` as a webhook in your GitHub repo settings, and enable **"GitHub hook trigger for GITScm polling"** on the Jenkins job.

### 8. Push and watch it run
```bash
git add .
git commit -m "Trigger pipeline"
git push
```

---

## 📸 Screenshots

*(Add screenshots here: Jenkins pipeline stage view, `kubectl get pods` output, and the running app)*

---

## 🧠 What I Learned

This project was built stage by stage, manually first and automated second, specifically to understand *why* each piece exists — not just to get a pipeline running. Along the way I worked through real issues including:

- Case-sensitive filenames and repo structure breaking Jenkins' SCM checkout
- Docker socket permissions and installing the Docker CLI inside a running container
- Cross-container Docker networking between Jenkins and Minikube
- Kubernetes authentication via kubeconfig, including Windows-to-Linux path translation issues
- The difference between `docker run` restarts and proper Kubernetes rolling updates (`kubectl set image` + `rollout status`)

A full write-up of every stage, including the errors and how they were debugged, is available here: **[Medium blog series](#)**

---

## 🔭 Roadmap

- [ ] Add a `production` namespace with a manual approval gate before deployment
- [ ] Workspace cleanup (`post { always { cleanWs() } }`)
- [ ] Image vulnerability scanning with Trivy
- [ ] Code quality checks with SonarQube
- [ ] Move off ngrok to a cloud-hosted Jenkins instance with a stable public address

---

## 📄 License

This project is for educational purposes. Feel free to fork it and use it as a learning template for your own DevOps journey.
