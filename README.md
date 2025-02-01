# Spring Boot Microservice on Kubernetes

This project is a **Spring Boot** microservice that runs inside **Kubernetes (Minikube)** and is deployed using **Docker & Skaffold**. It includes **Spring Cloud** features for service discovery (Eureka) and supports hot reloading for fast development.

## 🚀 Features
- **Spring Boot REST API** (`/api/hello`)
- **Containerized with Docker**
- **Deployed to Kubernetes using Minikube**
- **Skaffold for CI/CD & Hot Reloading**
- **Spring Cloud Eureka for Service Discovery**
- **Supports macOS (Apple Silicon - ARM64)**

---

## 📂 Project Structure
```
spring-k8s-demo/
│── src/                    # Spring Boot Source Code
│── k8s/                    # Kubernetes Manifests
│   ├── deployment.yaml      # Deployment Config
│   ├── service.yaml         # Service Config
│── Dockerfile               # Docker Configuration
│── skaffold.yaml            # Skaffold Config for CI/CD
│── README.md                # Project Documentation
```

---

## 🛠️ **Prerequisites**
Before you begin, make sure you have the following installed:
- **Java 23** (Eclipse Temurin / Amazon Corretto)
- **Docker** (for building images)
- **Minikube** (for local Kubernetes cluster)
- **kubectl** (Kubernetes CLI)
- **Skaffold** (for hot reloading)

---

## 🏗 **Setup & Installation**
### **1️⃣ Clone the Repository**
```sh
git clone https://github.com/your-repo/spring-k8s-demo.git
cd spring-k8s-demo
```

### **2️⃣ Start Minikube**
```sh
minikube start --profile custom --driver=docker
```
If Minikube is already running, make sure it's using the **correct Docker daemon**:
```sh
eval $(minikube -p custom docker-env)
```

### **3️⃣ Build and Load Docker Image into Minikube**
Since you are on **Apple Silicon (ARM64)**, build for `arm64`:
```sh
docker build --platform linux/arm64 -t spring-k8s-demo .
minikube image load spring-k8s-demo -p custom
```

### **4️⃣ Deploy to Kubernetes**
```sh
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### **5️⃣ Get Service URL**
```sh
minikube service spring-k8s-demo-service --url -p custom
```
✅ **Expected Output**
```
http://127.0.0.1:51557
```
Open this in your **browser** or test with `curl`:
```sh
curl http://127.0.0.1:51557/api/hello
```
✅ **Expected Response**
```
Hello from Spring Boot on Kubernetes!
```

---

## 🔥 **Using Skaffold for Continuous Development**
Skaffold watches for **code changes and redeploys automatically**.

### **1️⃣ Start Skaffold in Development Mode**
```sh
skaffold dev
```
- **Edit `src/main/java/com/example/demo/DemoApplication.java`**
- Change the response:
  ```java
  return "Hello from Kubernetes v2!";
  ```
- Save the file and **watch Skaffold automatically redeploy**! 🚀

### **2️⃣ Stop Skaffold**
Press:
```
Ctrl + C
```
Skaffold will **clean up all Kubernetes resources**.

---

## 📜 **Kubernetes Manifests**
### **Deployment (`k8s/deployment.yaml`)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-k8s-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-k8s-demo
  template:
    metadata:
      labels:
        app: spring-k8s-demo
    spec:
      containers:
        - name: spring-k8s-demo
          image: spring-k8s-demo:latest
          imagePullPolicy: Never  # Uses the locally built image
          ports:
            - containerPort: 8080
```

### **Service (`k8s/service.yaml`)**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: spring-k8s-demo-service
spec:
  selector:
    app: spring-k8s-demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: NodePort
```

---

## ⚙ **Troubleshooting**
### **1️⃣ Minikube Can't Find the Image**
```sh
minikube image load spring-k8s-demo -p custom
```

### **2️⃣ Pod Stuck in `ImagePullBackOff`**
Modify `k8s/deployment.yaml` to **disable pulling from Docker Hub**:
```yaml
imagePullPolicy: Never
```
Then apply the fix:
```sh
kubectl apply -f k8s/deployment.yaml
kubectl rollout restart deployment spring-k8s-demo
```

### **3️⃣ Port Forwarding Issue (macOS Docker Driver)**
If `minikube service` stops working when the terminal is closed, run:
```sh
nohup minikube service spring-k8s-demo-service -p custom --url > minikube-url.log 2>&1 &
```

---