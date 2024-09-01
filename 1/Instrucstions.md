**Challenge Scenario: Deploy a Real Web Application**

#### **Objective:**
You will deploy a web application that consists of a backend API service and a frontend service. The backend will serve JSON data, and the frontend will display this data. Both services will be deployed on Kubernetes.

#### **Tasks:**

1. **Create a Namespace**
   - Create a namespace for the application.

2. **Deploy the Backend**
   - Use a real backend Docker image (e.g., `ealen/echo-server`) that serves JSON data.
   - Expose the backend using a ClusterIP service.

3. **Deploy the Frontend**
   - Use a real frontend Docker image (e.g., `nginx`) to serve static content, with some modifications to make it communicate with the backend.
   - Expose the frontend using a LoadBalancer service (or NodePort if LoadBalancer is not available).

4. **Test the Application**
   - Ensure that the frontend can successfully retrieve and display data from the backend.

5. **Clean Up**
   - Clean up all resources created in this challenge.

### **Step-by-Step Instructions and Answers**

#### **1. Create a Namespace**
- **Task**: Create a namespace called `k8s-training`.

  ```bash
  kubectl create namespace k8s-training
  ```

- **Answer**: The command will create a namespace named `k8s-training`.

  ```bash
  kubectl get namespaces
  ```

#### **2. Deploy the Backend**

- **Task**: Deploy a simple backend API using the Docker image `ealen/echo-server`. This server will echo back JSON data that it receives.

  **Create Deployment**:
  Create a file named `backend-deployment.yaml`:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: backend
    namespace: k8s-training
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: backend
    template:
      metadata:
        labels:
          app: backend
      spec:
        containers:
        - name: backend
          image: ealen/echo-server
          ports:
            - containerPort: 8080
  ```

  Apply the Deployment:
  ```bash
  kubectl apply -f backend-deployment.yaml
  ```

  **Create Service**:
  Create a file named `backend-service.yaml`:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: backend-service
    namespace: k8s-training
  spec:
    selector:
      app: backend
    ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
    type: ClusterIP
  ```

  Apply the Service:
  ```bash
  kubectl apply -f backend-service.yaml
  ```

- **Answer**: The backend is now running and accessible within the cluster via `backend-service`.

  ```bash
  kubectl get pods -n k8s-training
  kubectl get svc -n k8s-training
  ```

#### **3. Deploy the Frontend**

- **Task**: Deploy a simple frontend service using the Docker image `nginx`. Modify the frontend to make a request to the backend and display the returned JSON data.

  **Create ConfigMap for Frontend Configuration**:
  Create a ConfigMap to serve a custom `index.html` file.

  ```bash
  kubectl create configmap frontend-config --from-file=index.html -n k8s-training
  ```

  Example `index.html`:
  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Frontend App</title>
      <script>
          async function fetchData() {
              const response = await fetch('http://backend-service');
              const data = await response.json();
              document.getElementById('data').textContent = JSON.stringify(data, null, 2);
          }
          window.onload = fetchData;
      </script>
  </head>
  <body>
      <h1>Data from Backend:</h1>
      <pre id="data">Loading...</pre>
  </body>
  </html>
  ```

  **Create Deployment**:
  Create a file named `frontend-deployment.yaml`:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: frontend
    namespace: k8s-training
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: frontend
    template:
      metadata:
        labels:
          app: frontend
      spec:
        containers:
        - name: frontend
          image: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - name: config-volume
              mountPath: /usr/share/nginx/html
    volumes:
      - name: config-volume
        configMap:
          name: frontend-config
  ```

  Apply the Deployment:
  ```bash
  kubectl apply -f frontend-deployment.yaml
  ```

  **Create Service**:
  Create a file named `frontend-service.yaml`:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: frontend-service
    namespace: k8s-training
  spec:
    selector:
      app: frontend
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
    type: LoadBalancer
  ```

  Apply the Service:
  ```bash
  kubectl apply -f frontend-service.yaml
  ```

- **Answer**: The frontend is now accessible externally. You can retrieve the external IP (if using LoadBalancer) or NodePort for access.

  ```bash
  kubectl get svc -n k8s-training
  ```

#### **4. Test the Application**

- **Task**: Access the frontend service in your browser. The page should display the JSON data returned from the backend.

  **Expected Result**:
  - When you open the frontend service in a web browser, it should display the JSON data returned from the backend API.

  **Testing Command** (if you don't have a LoadBalancer):
  ```bash
  minikube service frontend-service -n k8s-training
  ```

#### **5. Clean Up**

- **Task**: Clean up all resources created in this challenge.

  ```bash
  kubectl delete namespace k8s-training
  ```

- **Answer**: All resources associated with the `k8s-training` namespace are deleted.

### **Conclusion**

By completing this extended challenge, you will gain practical experience in deploying a real-world Kubernetes application that consists of a frontend and a backend service. This includes managing ConfigMaps, Services, Deployments, and more, which are essential skills for working with Kubernetes in production environments.

