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
