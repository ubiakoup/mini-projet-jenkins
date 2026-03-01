# 🚀 PayMyBuddy - CI/CD Pipeline with Jenkins

<img width="1536" height="1024" alt="architecture_paymybuddy" src="https://github.com/user-attachments/assets/89163478-0caf-4628-a3ff-2bd8c1304414" />


## 📌 Description

This project implements a complete **CI/CD pipeline using Jenkins** to deploy a **Spring Boot application** on AWS EC2 instances.

The pipeline automates:
* ✅ Automated testing (unit + integration)
* ✅ Code quality analysis with SonarCloud
* ✅ Docker build & push
* ✅ Deployment to Staging & Production
* ✅ Post-deployment validation tests
* ✅ Slack notifications
---

## 🏗️ Tech Stack

* **Backend**: Spring Boot
* **Build Tool**: Maven
* **CI/CD**: Jenkins (Declarative Pipeline)
* **Code Quality**: SonarCloud
* **Containerization**: Docker
* **Cloud**: AWS EC2
* **Database**: MySQL (Docker)
* **Notifications**: Slack

---

## 🔄 CI/CD Pipeline

### 📍 Main Stages

### 1️⃣ Tests + SonarCloud

* Runs unit and integration tests
* Performs static code analysis

```bash
mvn clean verify sonar:sonar
```

✔ Detects:

* bugs
* code smells
* vulnerabilities
* test coverage

---

### 2️⃣ Build & Push Docker Image

* Builds Docker image
* Pushes image to DockerHub

```bash
docker build -t <dockerhub_user>/paymybuddy:latest .
docker push <dockerhub_user>/paymybuddy:latest
```

---

### 3️⃣ Staging Deployment

On the staging EC2 instance:

* Creates Docker network
* Starts MySQL container with persistent volume
* Injects SQL initialization scripts (`create.sql`, `data.sql`)
* Deploys the application container

---

### 4️⃣ Production Deployment

Same process as staging but with:

* isolated environment
* dedicated database

---

### 5️⃣ Deployment Validation Tests

Automated validation after deployment:

```bash
curl http://<EC2_IP>:8081/login
```

✔ Verifies:

* application availability
* expected HTML content (`Pay My Buddy`)

---

### 6️⃣ Slack Notification

At the end of the pipeline:

* SUCCESS ✅
* FAILURE ❌

Automatically sent using:

```groovy
post {
  always {
    slackNotifier currentBuild.result
  }
}
```

---

## 🌿 Gitflow Strategy

### 🔹 `main` branch

Full pipeline execution:

* Tests
* SonarCloud
* Build & Push
* Staging Deployment
* Production Deployment
* Validation Tests

---

### 🔹 Other branches

Only CI steps are executed:

* Tests
* Code quality analysis
* Build (optional depending on configuration)

👉 No deployment for safety reasons

---

## 🐳 Database

MySQL is deployed using Docker with automatic initialization.

### 📁 Initialization scripts

```
src/main/resources/database/
├── create.sql
├── data.sql
```

### 📌 Mounted inside container

```bash
-v /home/ubuntu/init-db:/docker-entrypoint-initdb.d
```

✔ Automatically creates tables
✔ Inserts initial data

---

## 🔐 Credentials Management

Stored securely in Jenkins:

| Credential        | Purpose                   |
| ----------------- | ------------------------- |
| sonar_token       | SonarCloud authentication |
| dockerhub         | DockerHub login           |
| ssh_key           | EC2 SSH access            |
| DB_USER / DB_PASS | Database credentials      |

Usage example:

```groovy
SPRING_DATASOURCE_USERNAME = credentials('DB_USER')
```

---

## 🚀 Automated Deployment

### 📡 SSH from Jenkins

```bash
ssh ubuntu@<EC2_IP>
```

### 📦 Copy SQL files

```bash
scp src/main/resources/database/*.sql ubuntu@<EC2_IP>:/home/ubuntu/init-db/
```

---

## ⚠️ Issues Faced & Solutions

| Issue                  | Solution                   |
| ---------------------- | -------------------------- |
| mvnw permission denied | chmod +x mvnw              |
| docker not found       | mount docker.sock          |
| SonarCloud error       | disable automatic analysis |
| MySQL not reachable    | use docker network + wait  |
| EOF error              | fix heredoc syntax         |
| container crash        | fix DB configuration       |

---

## 🔥 Project Highlights

* ✔ Full CI/CD pipeline
* ✔ Multi-environment deployment (staging & production)
* ✔ Automated post-deployment testing
* ✔ Secure credential management (Jenkins)
* ✔ SonarCloud integration
* ✔ Slack notifications
* ✔ Fully containerized architecture

## 👨‍💻 Author

**Ulrich Kouatang**
IT/OT integrator 

