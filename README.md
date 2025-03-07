# **Cloud Native Resource Monitoring Python App on K8s!**

## Things you will Learn 🤯

1. Python and how to create a monitoring application in Python using Flask and psutil.
2. How to run a Python app locally.
3. Learn Docker and how to containerize a Python application:
    1. Creating a Dockerfile.
    2. Building a Docker image.
    3. Running a Docker container.
    4. Docker commands.
4. Create an ECR repository using Python Boto3 and push a Docker image to ECR.
5. Learn Kubernetes and create an EKS cluster and node groups.
6. Create Kubernetes deployments and services using Python.


## **Prerequisites** 📝

(Things to have before starting the project)

- [x] AWS Account.
- [x] Programmatic access and AWS configured with CLI.
- [x] Python3 installed.
- [x] Docker and Kubectl installed.
- [x] Code editor (VSCode).

# ✨ Let’s Start the Project ✨

## **Part 1: Deploying the Flask Application Locally**

### **Step 1: Clone the Code**

Clone the code from your repository:

```
git clone <repository_url>
```

### **Step 2: Install Dependencies**

The application uses the **`psutil`**, **`Flask`**, **`Plotly`**, and **`Boto3`** libraries. Install them using pip:

```
pip3 install -r requirements.txt
```

### **Step 3: Run the Application**

Navigate to the root directory of the project and execute the following command:

```
python3 app.py
```

This will start the Flask server on **`localhost:5000`**. Open [http://localhost:5000/](http://localhost:5000/) in your browser to access the application.

## **Part 2: Dockerizing the Flask Application**

### **Step 1: Create a Dockerfile**

Create a **`Dockerfile`** in the root directory of the project with the following contents:

```
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt .

# Install dependencies
RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is run
CMD ["flask", "run"]
```

### **Step 2: Build the Docker Image**

To build the Docker image, execute the following command:

```
docker build -t <image_name> .
```

### **Step 3: Run the Docker Container**

To run the Docker container, execute the following command:

```
docker run -p 5000:5000 <image_name>
```

The Flask server will now be accessible at **`localhost:5000`**. Open [http://localhost:5000/](http://localhost:5000/) in your browser.

## **Part 3: Pushing the Docker Image to ECR**

### **Step 1: Create an ECR Repository**

Create an ECR repository using Python:

```python
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```

### **Step 2: Push the Docker Image to ECR**

Push the Docker image to ECR:

```
docker tag <image_name> <ecr_repo_uri>:<tag>
docker push <ecr_repo_uri>:<tag>
```

## **Part 4: Creating an EKS Cluster and Deploying the App Using Python**

### **Step 1: Create an EKS Cluster**

Create an EKS cluster and add a node group.

### **Step 2: Create Deployment and Service**

```python
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="<your-ecr-repo-uri>:latest",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```

Make sure to replace the image URI on line 25 with your ECR image URI.

Once you run this file by executing `python3 eks.py`, the deployment and service will be created.

- Check the deployment, service, and pods:

```bash
kubectl get deployment -n default  # Check deployments
kubectl get service -n default     # Check services
kubectl get pods -n default        # Check pods
```

Once your pod is up and running, expose the service:

```bash
kubectl port-forward service/my-flask-service 5000:5000
```

Access the application on **`localhost:5000`**.

