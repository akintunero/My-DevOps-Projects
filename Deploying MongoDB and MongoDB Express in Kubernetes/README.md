

This DevOps project demonstrates how to deploy MongoDB and MongoDB Express in a Kubernetes cluster using YAML configurations. MongoDB is a NoSQL database, and MongoDB Express is a web-based administrative interface for MongoDB.



Before you begin, make sure you have the following prerequisites set up:

- A Kubernetes cluster provisioned and `kubectl` configured.
- Docker installed (if you need to build custom Docker images).
- Base64-encoded credentials for MongoDB (for creating a Kubernetes Secret).

## Deployment

### 1. Deploy MongoDB

Deploy MongoDB with 3 replicas using the following YAML configuration:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:latest
          ports:
            - containerPort: 27017
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-password

This configuration (mongo-deployment.yaml) specifies the MongoDB deployment with necessary labels, ports, and environment variables.
2. Expose MongoDB with a Service

Create a Kubernetes Service to expose MongoDB using the following YAML configuration:



apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017

The mongodb-service.yaml file defines a service that allows access to the MongoDB pods.
3. Deploy MongoDB Express

Deploy MongoDB Express with the following YAML configuration:


apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
        - name: mongo-express
          image: mongo-express
          ports:
            - containerPort: 8081
          env:
            - name: ME_CONFIG_MONGODB_ADMINUSERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-username
            - name: ME_CONFIG_MONGODB_ADMINPASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-password
            - name: ME_CONFIG_MONGODB_SERVER
              value: mongodb-service  # Update to match the name of your MongoDB service

The mongo-express-deployment.yaml file deploys MongoDB Express with the required environment variables.
4. Expose MongoDB Express

Create a Kubernetes Service to expose MongoDB Express (LoadBalancer for external access):



apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer  # Use LoadBalancer for external exposure, or ClusterIP for internal use
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081

The mongo-express-service.yaml file sets up a service that provides access to the MongoDB Express web interface.
5. Configuration (Optional)

If additional configuration is needed, you can use the following YAML configuration to define a ConfigMap with key-value pairs:



apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service  # Update to match the name of your MongoDB service
  # Add more key-value pairs as needed

6. Secret for MongoDB Credentials

Ensure you have a Secret (mongodb-secret.yaml) with Base64-encoded MongoDB credentials:


apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret  # Replace with a descriptive name for your secret
type: Opaque  # This type allows you to store arbitrary key-value pairs
data:
  mongo-root-username: BASE64_ENCODED_USERNAME  # Replace with your Base64-encoded username
  mongo-root-password: BASE64_ENCODED_PASSWORD  # Replace with your Base64-encoded password

Replace the placeholders in mongodb-secret.yaml with your actual Base64-encoded MongoDB username and password.
Accessing MongoDB Express

    For internal access within the cluster, consider using minikube service mongo-express-service (assuming you are using Minikube):



minikube service mongo-express-service

<img width="1440" alt="Screenshot 2023-09-26 at 02 33 54" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/af48b2bf-ce75-4b03-bdc0-cd88ac9847e6">



