# Deploying a Python Flask App on AWS EC2

This guide will walk you through the process of deploying a Python Flask web application on an AWS EC2 instance.

## Prerequisites

Before you begin, ensure you have the following prerequisites:

- An AWS account with the necessary permissions to create and manage EC2 instances.
- AWS CLI installed and configured with access credentials.
- SSH key pair created for connecting to the EC2 instance.
- Basic knowledge of Python, Flask, and the command line.

## Getting Started

### Create the EC2 Instance

1. Log in to your AWS Management Console.
2. Navigate to the EC2 dashboard.
3. Click the "Launch Instance" button to create a new EC2 instance.
4. Launch the instance.

### SSH into the EC2 Instance

1. Once your instance is running, open your terminal and use SSH to connect to it:

  
       ssh -i /path/to/your/key.pem ec2-user@your-ec2-instance-ip
   
   Update and Install Dependencies
   Update the package list on the EC2 instance:

       sudo apt update
   
   Install Python 3 and pip:
   
       sudo apt install python3 python3-pip
   
   Create a directory for your Flask app and navigate to it:
   
       mkdir flask_app && cd flask_app

   Install Python 3 virtual environment:
   
       sudo apt install python3.10-venv
   
   Create a virtual environment:

       python3 -m venv venv
       source venv/bin/activate


   Create the Flask App
   Use a text editor like vim or nano to create your Flask app file, e.g., app.py:

   sudo vim app.py
 
       from flask import Flask
    
       app = Flask(__name__)
    
       @app.route('/')
       def hello_world():
           return 'Hello, World!'
    
       if __name__ == '__main__':
           app.run(host="0.0.0.0", port=5000)

       
   Create a requirements file and add Flask and Werkzeug:

   sudo vim requirements.txt
   
       Flask==2.0.1
       Werkzeug==2.0.2

   
   Install the Flask dependencies:

       pip install -r requirements.txt


   Run the Flask App
   Start the Flask app:

       python app.py

   
   Allow incoming traffic on port 5000 in your EC2 instance's security group settings.

   Access your Flask app using your EC2 instance's public IP and port 5000:


       http://your-ec2-instance-ip:5000

   
   Your Python Flask app should now be up and running on AWS EC2.
   

  <img width="1440" alt="Screenshot 2023-10-02 at 12 16 24" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/2425ce04-718c-4d84-b04c-c6f6773d455f">


