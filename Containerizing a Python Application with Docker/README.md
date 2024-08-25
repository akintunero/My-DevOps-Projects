# Containerizing a Python Application with Docker

This is a simple "Hello, World!" application written in Python and containerized using Docker. This repository demonstrates how to create a Dockerfile for a Python application and run it as a container.

## Prerequisites

Make sure you have the following installed on your system:

- [Docker](https://docs.docker.com/get-docker/)

## Getting Started

Follow the steps below to set up and run the application.

### Step 1: Clone the Repository

Clone this repository to your local machine using the following command:

    git clone https://github.com/akintunero/My-DevOps-Projects.git
    cd python-docker-hello-world

Step 2: Create the Python Application
Create a file named app.py and add the following code:

```
    # app.py
    print("Hello, World!")
```

Step 3: Create a Dockerfile
Create a file named Dockerfile in the same directory and add the following instructions:

```
    # Use the official Python image from the Docker Hub
    FROM python:3.9-slim
    
    # Set the working directory in the container
    WORKDIR /app
    
    # Copy the current directory contents into the container at /app
    COPY . /app
    
    # Run app.py when the container launches
    CMD ["python", "app.py"]
```



Step 4: Build the Docker Image
Build the Docker image using the following command:

```
    docker build -t python-hello-world .
```

This command builds the Docker image and tags it as python-hello-world.

Step 5: Run the Docker Container

Run the container using the following command:

```
    docker run --rm python-hello-world
```
You should see the output:


    Hello, World!

Step 6: Verify the Container
To verify that the container is running, list all running containers:

```
    docker ps -a
```

This command will show you a list of all containers, including those that have exited. You should see your python-hello-world container in the list.