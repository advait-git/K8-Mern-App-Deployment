# Line-by-Line Explanations of Kubernetes Configuration Files

## 1. mongo-app.yaml

```yaml
apiVersion: apps/v1  # Specifies the API version for Kubernetes resources, apps/v1 is used for Deployments
kind: Deployment  # Defines that this is a Deployment resource which manages a set of identical pods
metadata:  # Contains metadata about the Deployment
  name: mongo-deployment  # The name of the Deployment, used to reference it later
  labels:  # Labels are key-value pairs that are attached to objects
    app: mongo  # Label with key 'app' and value 'mongo', used for identification
spec:  # Specification of the desired behavior of the Deployment
  replicas: 1  # Number of identical pod copies to maintain (only 1 for MongoDB in this case since it's stateful)
  selector:  # Defines how the Deployment finds which Pods to manage
    matchLabels:  # Selector uses matchLabels to find Pods with matching labels
      app: mongo  # Selects Pods with label 'app: mongo'
  template:  # Template for the Pods that will be created by this Deployment
    metadata:  # Metadata for the Pods
      labels:  # Labels for the Pods
        app: mongo  # Each Pod gets label 'app: mongo', which matches the selector above
    spec:  # Specification for the Pod
      containers:  # List of containers in the Pod
      - name: mongo  # Name of the container
        image: mongo:6.0  # Docker image to use for this container (MongoDB version 6.0)
        ports:  # Ports to expose from the container
        - containerPort: 27017  # MongoDB standard port, exposes this port from the container
        env:  # Environment variables to set in the container
        - name: MONGO_INITDB_ROOT_USERNAME  # Environment variable for MongoDB root username
          valueFrom:  # Instead of hardcoding the value, get it from another resource
            secretKeyRef:  # Reference a Secret resource
              name: mongo-secret  # Name of the Secret resource
              key: mongo-user  # Key within the Secret that contains the value
        - name: MONGO_INITDB_ROOT_PASSWORD  # Environment variable for MongoDB root password
          valueFrom:  # Get value from another resource
            secretKeyRef:  # Reference a Secret resource
              name: mongo-secret  # Name of the Secret
              key: mongo-password  # Key within the Secret

---  # Separator between Kubernetes resources in the same YAML file

apiVersion: v1  # Core API version, used for Services
kind: Service  # Defines this as a Service resource which exposes pods to network traffic
metadata:  # Metadata about the Service
  name: mongo-service  # Name of the Service
spec:  # Specification of the Service
  selector:  # Defines which Pods the Service routes traffic to
    app: mongo  # Selects Pods with label 'app: mongo'
  ports:  # Port configuration
    - protocol: TCP  # Protocol used (TCP is standard)
      port: 27017  # Port exposed by the Service
      targetPort: 27017  # Port to forward to in the Pod (MongoDB port)
  # Note: This is a ClusterIP service (default) which means it's only accessible within the cluster
```

## 2. mongo-config.yaml

```yaml
apiVersion: v1  # Core API version, used for ConfigMaps
kind: ConfigMap  # Defines this as a ConfigMap resource for non-sensitive configuration data
metadata:  # Metadata about the ConfigMap
  name: mongo-config  # Name of the ConfigMap
data:  # The actual configuration data
  mongo-url: mongo-service  # Key-value pair: 'mongo-url' is the key, 'mongo-service' is the value
  # This stores the MongoDB service name so other pods can reference it by name instead of IP
```

## 3. secret.yaml

```yaml
apiVersion: v1  # Core API version, used for Secrets
kind: Secret  # Defines this as a Secret resource for storing sensitive data
metadata:  # Metadata about the Secret
  name: mongo-secret  # Name of the Secret
type: Opaque  # Type of Secret; Opaque is generic, arbitrary data
data:  # The actual secret data
  mongo-user: bW9uZ291c2Vy  # Key-value pair for MongoDB username (base64 encoded)
  # 'bW9uZ291c2Vy' is base64 for 'mongouser'
  mongo-password: bW9uZ29wYXNzd29yZA==  # Key-value pair for MongoDB password (base64 encoded)
  # 'bW9uZ29wYXNzd29yZA==' is base64 for 'mongopassword'
```

## 4. web-app.yaml

```yaml
apiVersion: apps/v1  # Specifies the API version for Kubernetes resources, apps/v1 is used for Deployments
kind: Deployment  # Defines that this is a Deployment resource
metadata:  # Contains metadata about the Deployment
  name: webapp-deployment  # The name of the Deployment
  labels:  # Labels for identification
    app: webapp  # Label with key 'app' and value 'webapp'
spec:  # Specification of the desired behavior of the Deployment
  replicas: 1  # Number of identical pod copies to maintain
  selector:  # Defines how the Deployment finds which Pods to manage
    matchLabels:  # Selector uses matchLabels to find Pods with matching labels
      app: webapp  # Selects Pods with label 'app: webapp'
  template:  # Template for the Pods that will be created
    metadata:  # Metadata for the Pods
      labels:  # Labels for the Pods
        app: webapp  # Each Pod gets label 'app: webapp'
    spec:  # Specification for the Pod
      containers:  # List of containers in the Pod
      - name: mongo-express  # Name of the container
        image: mongo-express:latest  # Docker image for MongoDB Express (web UI for MongoDB)
        ports:  # Ports to expose from the container
        - containerPort: 8081  # Mongo Express standard port
        env:  # Environment variables for the container
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME  # MongoDB admin username for Mongo Express
          valueFrom:  # Get value from another resource
            secretKeyRef:  # Reference a Secret
              name: mongo-secret  # Name of the Secret
              key: mongo-user  # Key within the Secret
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD  # MongoDB admin password for Mongo Express
          valueFrom:  # Get value from another resource
            secretKeyRef:  # Reference a Secret
              name: mongo-secret  # Name of the Secret
              key: mongo-password  # Key within the Secret
        - name: ME_CONFIG_MONGODB_SERVER  # MongoDB server URL for Mongo Express
          valueFrom:  # Get value from another resource
            configMapKeyRef:  # Reference a ConfigMap
              name: mongo-config  # Name of the ConfigMap
              key: mongo-url  # Key within the ConfigMap

---  # Separator between Kubernetes resources

apiVersion: v1  # Core API version, used for Services
kind: Service  # Defines this as a Service resource
metadata:  # Metadata about the Service
  name: webapp-service  # Name of the Service
spec:  # Specification of the Service
  type: NodePort  # Type of Service: NodePort makes the Service accessible on a static port on each Node
  selector:  # Defines which Pods the Service routes traffic to
    app: webapp  # Selects Pods with label 'app: webapp'
  ports:  # Port configuration
    - protocol: TCP  # Protocol used
      port: 8081  # Port exposed by the Service
      targetPort: 8081  # Port to forward to in the Pod (Mongo Express port)
      nodePort: 30100  # External port on all Nodes where this Service can be accessed
      # NodePort must be in range 30000-32767
```
