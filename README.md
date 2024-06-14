# Accuknox-assessment-
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME Wisecow

# Run app.py when the container launches
CMD ["python", "app.py"]
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
labels:
  app: wisecow
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
        - name: wisecow
          image: <your-container-registry>/wisecow-app:latest
          ports:
            - containerPort: 80
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
      targetPort: 80
  type: LoadBalancer
  name: CI/CD Pipeline
on:
  push:
    branches:
      - main
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Login to Container Registry
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      - name: Build Docker image
        run: docker build -t <your-container-registry>/wisecow-app:latest .
      - name: Push Docker image
        run: docker push <your-container-registry>/wisecow-app:latest
      - name: Deploy to Kubernetes
        run: kubectl apply -f wisecow-deployment.yaml
        env:
          KUBECONFIG: ${{ secrets.KUBE_CONFIG_DATA }}
          
