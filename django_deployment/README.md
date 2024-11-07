# Deploying a Django To-Do API with Kubernetes

This guide walks you through deploying a Django To-Do application on a local Kubernetes cluster. The deployment includes two main components:
- **Application Pod**: Runs both the Gunicorn/Django container and an Nginx container
- **Database Pod**: Runs a PostgreSQL database

### Repository and Image Links
- [GitHub Repo for Django To-Do List Files](https://github.com/Asiwomex/django-todolist)
- [DockerHub Repo for Docker Image](https://hub.docker.com/repository/docker/asiwomex/django-todolist/general) (Use tag: **v1.0**)

### Kubernetes Resources Used
- **Init Container**: For Django static file collection
- **ConfigMap**: For environment variable management
- **Secrets**: For storing sensitive data
- **EmptyDir Volume**: For static content
- **Persistent Volume and Claim**: For database storage

## Deployment Steps

### 1. Set Up the Kubernetes Cluster
- Verify the `kind` cluster is running:
  ```
  kind get clusters
  ```
- If no cluster is available, create one named `cluster1`:
  ```
  kind create cluster --name cluster1
  ```

### 2. Create and Set Namespace
- Create the **django** namespace:
  ```
  kubectl create namespace django
  ```
- Set **django** as the current namespace for all operations:
  ```
  kubectl config set-context --current --namespace=django
  ```

### 3. Encode Secrets
- Navigate to the **database** and **application** directories to encode secrets using `base64`. Example:
  ```bash
  echo -n 'postgresdb' | base64
  ```

### 4. Deploy the Database
In the **database** directory, apply the necessary YAML configurations:
```
kubectl apply -f secret.yml
kubectl apply -f storage.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml
```

### 5. Configure Database IP in Application ConfigMap
- Retrieve the database service IP and update the application `ConfigMap` accordingly:
  ```
  kubectl get services
  ```

### 6. Deploy the Application
In the **application** directory, apply the following YAML files:
```
kubectl apply -f secret.yaml
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 7. Run Database Migrations (Optional)
- Get the app pod name, access the container, and run migrations:
  ```
  kubectl get pods
  kubectl exec -it <pod-name> -- /bin/bash
  python manage.py makemigrations
  python manage.py migrate
  ```

### 8. Access the Django Application
Set up port forwarding to access the Django app from a browser:
```
kubectl port-forward service/app-service 8000:80
```
Open a browser and go to [http://localhost:8000](http://localhost:8000) to access the app.

### 9. Cleanup
To delete all resources:
```
kubectl delete all --all
```

## Troubleshooting

### Check and Create ConfigMap
1. Verify if `nginx-conf` ConfigMap exists:
   ```bash
   kubectl get configmap nginx-conf -n django
   ```
2. If missing, create the ConfigMap:
   ```
   kubectl create configmap nginx-conf --from-file=nginx.conf -n django
   ```

### Verify Pod Status
- Check the status of all pods:
  ```bash
  kubectl get pod -n django
  ```

