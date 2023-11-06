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
