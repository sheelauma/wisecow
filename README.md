# wisecow
Problem Statement 1: Containerization and Deployment of Wisecow Application on Kubernetes
Steps to Achieve:
1. Dockerization:
Dockerfile for Wisecow Application: A Dockerfile defines the steps to create a container image for the Wisecow application. Below is an example Dockerfile that can be used.
dockerfile
Copy code
# Use an official Python runtime as a base image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app
# Copy the application code into the container
COPY . /app

# Install the dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose the port the app runs on
EXPOSE 5000

# Define environment variables
ENV FLASK_APP=app.py

# Run the application
CMD ["flask", "run", "--host=0.0.0.0"]
This Dockerfile:
Uses Python 3.9 as the base image.
Copies the application code into the container.
Installs the required dependencies from requirements.txt.
Exposes port 5000 (assuming the app runs on this port).
Runs the Flask application.
2. Kubernetes Deployment:
Kubernetes Manifest Files: The Kubernetes manifest consists of multiple YAML files, such as the Deployment, Service, and Ingress resources.
yaml
Copy code
# wisecow-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
        spec:
      containers:
      - name: wisecow-container
        image: <container_registry>/wisecow:latest
        ports:
        - containerPort: 5000
---
# wisecow-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
---
# wisecow-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  rules:
  - host: wisecow.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wisecow-service
            port:
              number: 80
  tls:
  - hosts:
    - wisecow.example.com
    secretName: wisecow-tls-secret
The above manifest:
Defines a Deployment for the Wisecow app with 3 replicas.
Creates a Service to expose the app on port 80 (internally mapped to port 5000 of the container).
Defines an Ingress with TLS support for secure communication.
3. Continuous Integration and Deployment (CI/CD):
GitHub Actions Workflow: The workflow automates the process of building the Docker image, pushing it to a container registry (e.g., Docker Hub, GCR), and deploying it to Kubernetes.
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build Docker image
      run: |
        docker build -t wisecow-app .
        docker tag wisecow-app <container_registry>/wisecow:latest

    - name: Push Docker image to registry
      run: |
        docker push <container_registry>/wisecow:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up kubectl
      uses: actions/setup-kubectl@v2
      with:
        kubeconfig: ${{ secrets.KUBECONFIG }}

    - name: Deploy to Kubernetes
      run: |
        kubectl apply -f wisecow-deployment.yaml
        kubectl apply -f wisecow-service.yaml
        kubectl apply -f wisecow-ingress.yaml
The CI/CD pipeline:
Builds and pushes the Docker image whenever changes are pushed to the main branch.
Deploys the updated application to the Kubernetes cluster.
4. TLS Implementation:
To implement TLS, the Ingress resource will require a TLS certificate, which can be created using cert-manager or manually managed.

bash
Copy code
kubectl create secret tls wisecow-tls-secret --cert=path/to/cert.crt --key=path/to/cert.key
This command creates a Kubernetes Secret that stores the TLS certificate, which the Ingress resource uses to secure communication.
Problem Statement 2: System Health Monitoring and Application Health Checker Scripts
1. System Health Monitoring Script (Python)
2. import psutil
import smtplib
from email.mime.text import MIMEText

# Define threshold values
CPU_THRESHOLD = 80  # 80%
MEMORY_THRESHOLD = 80  # 80%
DISK_THRESHOLD = 80  # 80%

# Function to send an alert email
def send_alert(message):
    msg = MIMEText(message)
    msg['Subject'] = 'System Health Alert'
    msg['From'] = 'alert@example.com'
    msg['To'] = 'admin@example.com'

    with smtplib.SMTP('smtp.example.com') as server:
        server.sendmail('alert@example.com', 'admin@example.com', msg.as_string())

# Check CPU usage
cpu_usage = psutil.cpu_percent(interval=1)
if cpu_usage > CPU_THRESHOLD:
    send_alert(f"High CPU usage detected: {cpu_usage}%")

# Check memory usage
memory = psutil.virtual_memory()
if memory.percent > MEMORY_THRESHOLD:
    send_alert(f"High memory usage detected: {memory.percent}%")

# Check disk usage
disk = psutil.disk_usage('/')
if disk.percent > DISK_THRESHOLD:
    send_alert(f"High disk usage detected: {disk.percent}%")
This script checks CPU, memory, and disk usage, and sends an email alert if any metric exceeds the defined threshold.
2. Application Health Checker (Bash)
#!/bin/bash

# Define the application URL
URL="http://localhost:5000"

# Send a request to the application and capture the HTTP status code
HTTP_STATUS=$(curl -o /dev/null -s -w "%{http_code}" $URL)

# Check if the HTTP status is 200 (OK)
if [ $HTTP_STATUS -eq 200 ]; then
  echo "Application is up and running!"
else
  echo "Application is down! HTTP Status: $HTTP_STATUS"
fi
This script sends a request to the application and checks the HTTP status code. If it’s not 200 (OK), it indicates the application is down.
Conclusion:
The provided solution demonstrates how to containerize the Wisecow application, deploy it to Kubernetes with secure TLS communication, and implement an automated CI/CD pipeline using GitHub Actions. Additionally, I’ve provided two scripts for system health monitoring and application health checking to fulfill the objectives of Problem Statement 2.
