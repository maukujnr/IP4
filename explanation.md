# Explanation of a Docker Containerization Project -IP2

## Objective 1: Choice of Base Image

For the web application container, I chose the official Node.js image (node:14) as it's compatible with the application's stack.
I use smaller base images (node:14-slim and node:14-alpine) to reduce the image sizes.

Dockerfile for the web/client application container:
For the client, we copy only the build/ directory to the production image, which contains the compiled output. We will create a separate production build in the first stage and then copy the resulting build files to the second stage

```Dockerfile
# Use an official Node.js runtime as a build environment
FROM node:14-slim as build

# Set the working directory for the build stage
WORKDIR /app

# Copy package.json and package-lock.json for dependency installation
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the client application code to the working directory
COPY . .

# Build the client application
RUN npm run build

# Use a smaller base image for the production stage
FROM nginx:alpine

# Copy the build output from the build stage
COPY --from=build /app/build /usr/share/nginx/html

# Expose port 80
EXPOSE 80
```

For the backend application container, we install only production dependencies and copy only necessary files.
We expose the required ports (80 for the client, 5000 for the backend).

```Dockerfile
# Use a smaller base image with Node.js
FROM node:14-slim as build

# Set the working directory for the build stage
WORKDIR /app

# Copy package.json and package-lock.json for dependency installation
COPY package*.json ./

# Install dependencies
RUN npm install --production

# Copy the rest of the backend application code
COPY . .

# Use a smaller base image for the production stage
FROM node:14-alpine

# Set the working directory for the production stage
WORKDIR /app

# Copy only the necessary files from the build stage
COPY --from=build /app ./

# Expose the port (adjust to your application's port)
EXPOSE 5000
```

A docker-compose file is provided with the following contents:

```docker-compose.yml

version: '3'
services:
  client:
    build:
      context: client
    ports:
      - "80:3000"
    depends_on:
      - backend

  backend:
    build:
      context: backend
    ports:
      - "5000:5000"
    environment:
      MONGO_URI: mongodb://mongo:27017/yolomy
    depends_on:
      - mongo

  mongo:
    image: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:

```

## Getting Started

1. Clone this repository to your local machine.

```bash
git clone https://github.com/maukujnr/yolo.git

```
2. Navigate to the repository directory.
```bash
cd yolo
```
3. Run the application using Docker Compose.

```bash
docker-compose up -d
```
4. You can access the Yolo application in your web browser:

Client: http://localhost:3000
Backend: http://localhost:5000
MongoDB is already configured for the Backend container.


### IP3 Explanation
To run this application, a vagrant configuration and an ansibleplaybook is provided at the root of this repo. You can follow the following steps:
1. make sure you have Vagrant installed on your PC/MAC
2. Clone the Yolo repository:
```bash
git clone https://github.com/maukujnr/yolo.git
```
3. Change your working directory to the cloned repository:

```bash
cd yolo
```
4. Use Vagrant to start and provision the virtual machine:

```bash
vagrant up
```
#NB: When Vagrant is installed on the Windows system, the version installed within the Linux distribution must match.
By default Vagrant will not access features available on the Windows system from within the WSL. This means the VirtualBox and Hyper-V providers will not be available. To enable Windows access, which will also enable the VirtualBox and Hyper-V providers, set the ``VAGRANT_WSL_ENABLE_WINDOWS_ACCESS`` environment variable:
``
 export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
 ``