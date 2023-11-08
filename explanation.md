# Deploying Project Yolo on Kubernetes -IP4

## Intro

This application is implemnted using three components namely, a front-end (client), a backend (backed) and a database to help persist the data. 
This page provides a quick overview of how to deploy this application on Kubernetes. All the necessary files to successfully deploy on a kubernetes cluster are included in the kubernetes folder within this projects root directory.

## Choice of Kubernetes Objects

The choice of Kubernetes objects for deployment depends on the application's requirements and architecture. For this application we are using:

### Deployments for the backend and client components.
#### Why?

Deployments are suitable for stateless applications that can easily scale horizontally. A deployment will manage the creation and management of ReplicaSets and pods, ensuring high availability and updates without downtime.

#### frontend app deployment manifest
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-deployment  # Specify the name of the deployment
spec:
  replicas: 3  # Set the desired number of pod replicas
  selector:
    matchLabels:
      app: client  # Define the label selector to match pods with the label 'app: client'
  template:
    metadata:
      labels:
        app: client  # Label the pods created by this deployment with 'app: client'
    spec:
      containers:
      - name: client  # Assign a name to the container
        image: maukujnr/yolo-client-image:v1.1  # Specify the Docker image and version
        resources:
          requests:
            memory: "64Mi"  # Define the minimum memory request for the container
            cpu: "250m"  # Define the minimum CPU request for the container
          limits:
            memory: "128Mi"  # Set the maximum memory limit for the container
            cpu: "500m"  # Set the maximum CPU limit for the container
        ports:
        - containerPort: 3000  # Define the port on which the container will listen
```
#### backend app deployment manifest

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment  # Specify the name of the deployment
spec:
  replicas: 3  # Set the desired number of pod replicas
  selector:
    matchLabels:
      app: backend  # Define the label selector to match pods with the label 'app: backend'
  template:
    metadata:
      labels:
        app: backend  # Label the pods created by this deployment with 'app: backend'
    spec:
      containers:
      - name: backend  # Assign a name to the container
        image: maukujnr/yolo-backend-image:v1.1  # Specify the Docker image and version
        resources:
          requests:
            memory: "64Mi"  # Define the minimum memory request for the container
            cpu: "250m"  # Define the minimum CPU request for the container
          limits:
            memory: "128Mi"  # Set the maximum memory limit for the container
            cpu: "500m"  # Set the maximum CPU limit for the container
        ports:
        - containerPort: 8080  # Define the port on which the container will listen
```
### Statefulset for the database component.
#### Why?

StatefulSets are ideal for applications that require stable network identifiers and persistent storage. 

The use of StatefulSets is recommended for our application to ensure persistent storage. It will allow you to maintain a unique identity for each pod, which is essential for our database. 

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb-statefulset  # Name of the StatefulSet for MongoDB
spec:
  serviceName: mongodb  # Headless service for stable network identities
  replicas: 1  # Number of MongoDB pods
  selector:
    matchLabels:
      app: mongodb  # Label selector for MongoDB pods
  template:
    metadata:
      labels:
        app: mongodb  # Label for MongoDB pods
    spec:
      containers:
      - name: mongodb
        image: mongo:latest  # Docker image for MongoDB
        ports:
        - containerPort: 27017  # Port for MongoDB
        volumeMounts:
        - name: mongodb-storage
          mountPath: /data/db  # Mount path for MongoDB data
```

### The following section defines the Services for the frontend and backend apps
##### Front-end
```front-end app service
apiVersion: v1
kind: Service
metadata:
  name: client-service  # Specify the name of the service
spec:
  selector:
    app: client  # Define the label selector to route traffic to pods with the label 'app: client'
  ports:
    - protocol: TCP
      port: 80  # Define the port through which the service will be accessed
      targetPort: 3000  # Specify the target port on the pods where traffic will be forwarded
  type: LoadBalancer  # Use a LoadBalancer service type to expose the service to the internet
```
##### Backend
```backend app service
apiVersion: v1
kind: Service
metadata:
  name: backend-service  # Specify the name of the service
spec:
  selector:
    app: backend  # Define the label selector to route traffic to pods with the label 'app: backend'
  ports:
    - protocol: TCP
      port: 80  # Define the port through which the service will be accessed
      targetPort: 8080  # Specify the target port on the pods where traffic will be forwarded
  type: LoadBalancer  # Use a LoadBalancer service type to expose the service to the internet
```
##### Database
```
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service  # Specify the name of the service
spec:
  selector:
    app: mongodb  # Define the label selector to route traffic to pods with the label 'app: mongodb'
  ports:
    - protocol: TCP
      port: 27017  # Define the port through which the service will be accessed
      targetPort: 27017  # Specify the target port on the pods where traffic will be forwarded
  type: ClusterIP  # Use a ClusterIP service type for internal access within the cluster

  ```

### Storage Option

A Persistent Volume Claim (PVC) is is required so as to request a storage volume (PV) to store the database data.

A Persistent Volume (PV) provides a standardized way to manage and allocate storage resources in a Kubernetes cluster, ensuring that the applications have access to the storage they need in a reliable and consistent manner. It's claimed by pods using Persistent Volume Claims (PVCs).

### The following section defines the storage objects required by the app in this repo

  ```Persistent Volume Claim (PVC) for MongoDB data
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc  # Name of the PVC for MongoDB data
spec:
  accessModes:
    - ReadWriteOnce  # Access mode for the PVC
  resources:
    requests:
      storage: 1Gi  # Storage size for MongoDB data
  ```
  
  ```Persistent Volume (PV) for MongoDB data
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv  # Name of the PV for MongoDB data
spec:
  capacity:
    storage: 1Gi  # Storage size for the PV
  accessModes:
    - ReadWriteOnce  # Access mode for the PV
  persistentVolumeReclaimPolicy: Retain  # Reclaim policy for the PV
  hostPath:
    path: /your/host/path  # Host path for the PV

  ```
Please make sure to create the PV and PVC with appropriate settings on your own Kubernetes cluster. This will ensure your MongoDB data is stored persistently.