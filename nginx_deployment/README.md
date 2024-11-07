# Nginx Kubernetes Deployment and Service

This guide provides instructions to deploy an `nginx` web server on a Kubernetes cluster using a Deployment and Service YAML configuration. This setup will allow you to access the `nginx` server through a ClusterIP service, with an option to access it locally via port-forwarding.

## Prerequisites

- Kubernetes cluster set up and running.
- `kubectl` command-line tool configured to interact with your Kubernetes cluster.

## Configuration Files

The following files are included:

### 1. Deployment (`nginx-deployment`)

The deployment defines an `nginx` pod with:
- A single replica.
- The latest `nginx` image.
- An open port at 80 for web traffic.

### 2. Service (`nginx-service`)

The service creates an internal ClusterIP for accessing the `nginx` pod within the cluster:
- Exposes port 80, targeting the `nginx` container's port.
- Maps the deployment through a selector matching the `app: nginx` label.

## Deployment YAML
- The deployment yml file is named nginx_deploy.yml

### Port-Forwarding

To access the `nginx` service locally, set up port-forwarding from your local machine to the ClusterIP service:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Once port-forwarding is established, you can access `nginx` by visiting `http://localhost:8080` in your web browser.

## Deployment Steps

1. **Apply the YAML file to your cluster:**

   ```bash
   kubectl apply -f nginx.yml
   ```

2. **Verify the Deployment and Service:**

   Check if the deployment and service have been created successfully:

   ```bash
   kubectl get deployments -n savvyminds
   kubectl get services -n savvyminds
   ```

   > **Note**: If you removed the `namespace: savvyminds` lines, skip the `-n savvyminds` option.

3. **Port-Forward the Service (Optional):**

   If you want to test access from your local machine, run:

   ```bash
   kubectl port-forward service/nginx-service 8080:80
   ```

4. **Access Nginx Locally:**

   Open your web browser and go to:

   ```
   http://localhost:8080
   ```

   You should see the default `nginx` welcome page.

## Notes

- **Namespace**: If you don’t have a namespace named `savvyminds`, either create it using `kubectl create namespace savvyminds` or remove the `namespace` lines from the YAML to deploy to the default namespace.
- **Scaling**: To adjust the number of replicas, modify the `replicas` field in the deployment spec.

## Troubleshooting

- **Pod Issues**: Check pod status and logs if the deployment does not behave as expected:
  
  ```bash
  kubectl get pods -n savvyminds
  kubectl logs <pod-name> -n savvyminds
  ```

- **Service Access**: If `port-forward` doesn’t work as expected, verify the service and pod IPs and ports.

--- 

This setup should help you quickly deploy and access `nginx` on your Kubernetes cluster.