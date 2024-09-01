Certainly! Here’s another Kubernetes challenge designed to help you drill your skills. This challenge will focus on deploying a multi-tier application, incorporating secrets, environment variables, persistent storage, and basic network policies.

### **Challenge Scenario: Deploy a Multi-Tier Application**

#### **Objective:**
You will deploy a multi-tier web application on Kubernetes. The application consists of a MySQL database as the backend, a Python Flask application as the middle tier, and an Nginx server as the frontend. The Flask application will serve content from the database, and the Nginx server will serve static content while proxying API requests to the Flask application.

#### **Tasks:**

1. **Create a Namespace**
   - Create a namespace for the application.

2. **Deploy the MySQL Database**
   - Deploy a MySQL database with a PersistentVolume and a PersistentVolumeClaim.
   - Use a Kubernetes Secret to store MySQL credentials.

3. **Deploy the Flask Application**
   - Deploy a Python Flask application that connects to the MySQL database.
   - Pass the database credentials to the Flask application using environment variables.

4. **Deploy the Nginx Frontend**
   - Deploy an Nginx server that serves static content and proxies API requests to the Flask application.

5. **Configure Network Policies**
   - Apply a basic NetworkPolicy to restrict access to the MySQL database, allowing only the Flask application to connect.

6. **Test the Application**
   - Ensure the Nginx frontend is accessible, and it correctly serves static content and API responses.

7. **Clean Up**
   - Clean up all resources created in this challenge.

### **Step-by-Step Instructions and Answers**

#### **1. Create a Namespace**
- **Task**: Create a namespace called `multi-tier-app`.

  ```bash
  kubectl create namespace multi-tier-app
  ```

- **Answer**: The command will create a namespace named `multi-tier-app`.

  ```bash
  kubectl get namespaces
  ```

#### **2. Deploy the MySQL Database**

- **Task**: Deploy MySQL with persistent storage and credentials stored in a Secret.

  **Create Secret for MySQL Credentials**:
  ```bash
  kubectl create secret generic mysql-secret --from-literal=MYSQL_ROOT_PASSWORD=rootpassword --from-literal=MYSQL_DATABASE=flaskdb --from-literal=MYSQL_USER=flaskuser --from-literal=MYSQL_PASSWORD=flaskpassword -n multi-tier-app
  ```

  **Create PersistentVolume and PersistentVolumeClaim**:
  Create a file named `mysql-pv-pvc.yaml`:
  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: mysql-pv
    namespace: multi-tier-app
  spec:
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: /mnt/mysql-data
  ---
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: mysql-pvc
    namespace: multi-tier-app
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  ```

  Apply the PV and PVC:
  ```bash
  kubectl apply -f mysql-pv-pvc.yaml
  ```

  **Create MySQL Deployment and Service**:
  Create a file named `mysql-deployment.yaml`:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mysql
    namespace: multi-tier-app
  spec:
    selector:
      matchLabels:
        app: mysql
    replicas: 1
    template:
      metadata:
        labels:
          app: mysql
      spec:
        containers:
        - name: mysql
          image: mysql:5.7
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_DATABASE
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_USER
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
        volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: mysql-service
    namespace: multi-tier-app
  spec:
    ports:
      - port: 3306
    selector:
      app: mysql
  ```

  Apply the MySQL Deployment and Service:
  ```bash
  kubectl apply -f mysql-deployment.yaml
  ```

- **Answer**: The MySQL database is now running with persistent storage and is accessible within the cluster via `mysql-service`.

  ```bash
  kubectl get pods -n multi-tier-app
  kubectl get svc -n multi-tier-app
  ```

#### **3. Deploy the Flask Application**

- **Task**: Deploy a Python Flask application that connects to the MySQL database.

  **Create Flask Deployment and Service**:
  Create a file named `flask-deployment.yaml`:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: flask-app
    namespace: multi-tier-app
  spec:
    selector:
      matchLabels:
        app: flask-app
    replicas: 2
    template:
      metadata:
        labels:
          app: flask-app
      spec:
        containers:
        - name: flask-app
          image: tiangolo/uwsgi-nginx-flask:python3.8
          ports:
            - containerPort: 80
          env:
            - name: MYSQL_HOST
              value: "mysql-service"
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_DATABASE
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_USER
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: flask-service
    namespace: multi-tier-app
  spec:
    ports:
      - port: 80
    selector:
      app: flask-app
  ```

  Apply the Flask Deployment and Service:
  ```bash
  kubectl apply -f flask-deployment.yaml
  ```

- **Answer**: The Flask application is now running and is accessible within the cluster via `flask-service`.

  ```bash
  kubectl get pods -n multi-tier-app
  kubectl get svc -n multi-tier-app
  ```

#### **4. Deploy the Nginx Frontend**

- **Task**: Deploy an Nginx server that serves static content and proxies API requests to the Flask application.

  **Create ConfigMap for Nginx Configuration**:
  Create a custom Nginx configuration using a ConfigMap.

  Create a file named `nginx-configmap.yaml`:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: nginx-config
    namespace: multi-tier-app
  data:
    default.conf: |
      server {
          listen 80;
          server_name localhost;

          location / {
              root /usr/share/nginx/html;
              index index.html;
          }

          location /api/ {
              proxy_pass http://flask-service;
          }
      }
  ```

  Apply the ConfigMap:
  ```bash
  kubectl apply -f nginx-configmap.yaml
  ```

  **Create Nginx Deployment and Service**:
  Create a file named `nginx-deployment.yaml`:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx
    namespace: multi-tier-app
  spec:
    selector:
      matchLabels:
        app: nginx
    replicas: 2
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config-volume
              mountPath: /etc/nginx/conf.d
            - name: nginx-static-volume
              mountPath: /usr/share/nginx/html
        volumes:
        - name: nginx-config-volume
          configMap:
            name: nginx-config
        - name: nginx-static-volume
          emptyDir: {}
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
    namespace: multi-tier-app
  spec:
    type: LoadBalancer
    ports:
      - port: 80
    selector:
      app: nginx
  ```

  Apply the Nginx Deployment and Service:
  ```bash
  kubectl apply -f nginx-deployment.yaml
  ```

- **Answer**: The Nginx server is now running and is accessible externally via `nginx-service`.

  ```bash
  kubectl get svc -n multi-tier-app
  ```

#### **5. Configure Network Policies**

- **Task**: Apply a

 NetworkPolicy to restrict access to the MySQL database, allowing only the Flask application to connect.

  Create a file named `mysql-networkpolicy.yaml`:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: mysql-network-policy
    namespace: multi-tier-app
  spec:
    podSelector:
      matchLabels:
        app: mysql
    ingress:
    - from:
      - podSelector:
          matchLabels:
            app: flask-app
      ports:
      - protocol: TCP
        port: 3306
  ```

  Apply the NetworkPolicy:
  ```bash
  kubectl apply -f mysql-networkpolicy.yaml
  ```

- **Answer**: The NetworkPolicy is now applied, restricting access to the MySQL database to only the Flask application.

  ```bash
  kubectl get networkpolicy -n multi-tier-app
  ```

#### **6. Test the Application**

- **Task**: Ensure the Nginx frontend is accessible, and it correctly serves static content and API responses.

  **Testing Command** (if you don't have a LoadBalancer):
  ```bash
  minikube service nginx-service -n multi-tier-app
  ```

  **Expected Result**:
  - Accessing the Nginx service should show the static content, and accessing `/api/` should correctly proxy requests to the Flask application, returning data from the MySQL database.

#### **7. Clean Up**

- **Task**: Clean up all resources created in this challenge.

  ```bash
  kubectl delete namespace multi-tier-app
  ```

- **Answer**: All resources associated with the `multi-tier-app` namespace are deleted.

### **Conclusion**

This challenge gives you hands-on experience with deploying a multi-tier application on Kubernetes. It covers essential Kubernetes concepts such as PersistentVolumes, Secrets, ConfigMaps, Services, Deployments, NetworkPolicies, and LoadBalancers. By completing this challenge, you will have a deeper understanding of how to manage and secure multi-tier applications in a Kubernetes environment.



