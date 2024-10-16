
# Node.js and Nexus Integration Project

This project demonstrates how to integrate a Node.js "Hello World" application with a Nexus Repository Manager.

## Prerequisites

- Docker and Nexus Repository Manager installed
- Node.js and npm installed

## Step 1: Set Up Nexus as an npm Repository

1. **Access Nexus Repository Manager**:
   Open your web browser and navigate to `http://<your-server-ip>:8081`.
![alt text](<Screenshot 2024-11-03 at 15.25.36-1.png>)

2. **Log in to Nexus** with the admin credentials (default password found in `/nexus-data/admin.password`).

3. **Create an npm hosted repository**:
   - Navigate to **Repositories**.
   - Click on **Create repository**.
   - Select **npm (hosted)**.
   - Configure the repository with a suitable name, e.g., `npm-hosted`.
   - Click **Create repository**.

4. **Create an npm proxy repository** (optional but recommended to proxy public npm registry):
   - Navigate to **Repositories**.
   - Click on **Create repository**.
   - Select **npm (proxy)**.
   - Configure the repository with a suitable name, e.g., `npm-proxy`, and set the remote storage URL to `https://registry.npmjs.org/`.
   - Click **Create repository**.

5. **Create an npm group repository**:
   - Navigate to **Repositories**.
   - Click on **Create repository**.
   - Select **npm (group)**.
   - Configure the repository with a suitable name, e.g., `npm-group`.
   - Add `npm-hosted` and `npm-proxy` (if created) to the group members.
   - Click **Create repository**.

6. **Get the Repository URL**:
   - Note down the URL of the `npm-group` repository. It will be something like `http://<your-server-ip>:8081/repository/npm-group/`.

## Step 2: Configure Your Node.js Project to Use Nexus

1. **Create a Node.js "Hello World" Project**:

   ```
   mkdir hello-world
   cd hello-world
   npm init -y
   ```

   Create an `index.js` file:

   ```
   // index.js
   console.log('Hello, World!');
   ```

2. **Configure npm to Use Nexus**:
   Configure npm to use your Nexus npm group repository:

   ```
   npm set registry http://<your-server-ip>:8081/repository/npm-group/
   ```

3. **Authenticate npm (if your Nexus repository requires authentication)**:
   - Create a user in Nexus if you haven't already.
   - Generate an npm token:

     ```
     npm login --registry=http://<your-server-ip>:8081/repository/npm-group/
     ```

   - Enter your Nexus username, password, and email when prompted.

## Step 3: Publish and Install Packages

1. **Publish a Package to Nexus**:
   Create a simple package and publish it to your Nexus npm-hosted repository:

   ```
   mkdir my-package
   cd my-package
   npm init -y
   ```

   Create an `index.js` file:

   ```
   // index.js
   module.exports = () => {
       console.log('Hello from my-package!');
   };
   ```

   Update `package.json` with the following:

   ```
   {
     "name": "my-package",
     "version": "1.0.0",
     "main": "index.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     "author": "",
     "license": "ISC"
   }
   ```

   Publish the package to Nexus:

   ```
   npm publish --registry=http://<your-server-ip>:8081/repository/npm-hosted/
   ```

2. **Install the Package in Your Node.js Project**:
   Go back to your "Hello World" project and install the package from Nexus:

   ```
   cd ../hello-world
   npm install my-package --registry=http://<your-server-ip>:8081/repository/npm-group/
   ```

3. **Use the Installed Package**:
   Modify `index.js` to use the installed package:

   ```
   // index.js
   const myPackage = require('my-package');
   myPackage();
   console.log('Hello, World!');
   ```

4. **Run Your Project**:

   ```
   node index.js
   ```

   You should see the output:

   ```
   Hello from my-package!
   Hello, World!
   ```


