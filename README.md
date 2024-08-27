# Kubernetes Learning Journey Documentation with Minikube

### 1. **Introduction to Kubernetes:**
   - **Definition:** Kubernetes is an open-source platform designed to automate the deployment, scaling, and operation of application containers across clusters of hosts. It provides a container-centric management environment.
   - **Key Concepts:**
      - *Cluster:* A set of nodes (machines) that run containerized applications managed by Kubernetes.
      - *Node:* A single machine (virtual or physical) within a Kubernetes cluster that runs containerized applications.
      - *Pod:* The smallest and simplest Kubernetes object. A Pod represents a single instance of a running process in a cluster and can contain one or more containers.
      - *Service:* An abstraction that defines a logical set of Pods and a policy by which to access them, such as load balancing.
      - *Namespace:* A way to divide cluster resources between multiple users (via resource quotas).
      - *Deployment:* A resource object in Kubernetes that provides declarative updates to applications, defining the desired state of Pods and ReplicaSets.
      - *ReplicaSet:* Ensures that a specified number of pod replicas are running at any given time.
      - *ConfigMap:* A Kubernetes object that allows you to decouple environment-specific configuration from your containerized applications.
      - *Secret:* Similar to ConfigMap, but intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys.
      - *Ingress:* Manages external access to the services in a cluster, typically HTTP.
      - *Kubelet:* The primary node agent that runs on each node in the Kubernetes cluster. It ensures that containers are running in a Pod.
      - *Kubectl:* The command-line tool for interacting with the Kubernetes cluster.
      - **Minikube:** A tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a virtual machine on your personal computer.

### 2. **Setting Up Minikube:**
   - **Installation:**
      - Install Minikube on your operating system by following the instructions on the [official Minikube documentation](https://minikube.sigs.k8s.io/docs/start/).
      - Install Kubectl, the command-line tool for Kubernetes, from the [Kubernetes official website](https://kubernetes.io/docs/tasks/tools/).
   - **Starting Minikube:**
      - Start a Kubernetes cluster using Minikube:
         ```bash
         minikube start
         ```
      - Verify the status of your Minikube cluster:
         ```bash
         minikube status
         ```
   - **Basic Minikube Commands:**
      - `minikube start`: Starts a local Kubernetes cluster.
      - `minikube stop`: Stops the Minikube cluster.
      - `minikube delete`: Deletes the Minikube cluster.
      - `minikube dashboard`: Accesses the Kubernetes dashboard for managing the cluster.

### 3. **Basic Kubernetes Commands with Minikube:**
   - **Managing Pods:**
      - Create a simple Pod using YAML or directly with `kubectl`:
         ```bash
         kubectl run nginx-pod --image=nginx --restart=Never
         ```
      - List all Pods in the cluster:
         ```bash
         kubectl get pods
         ```
      - Delete a Pod:
         ```bash
         kubectl delete pod nginx-pod
         ```
   - **Working with Deployments:**
      - Create a Deployment:
         ```bash
         kubectl create deployment nginx-deployment --image=nginx
         ```
      - Scale the Deployment to run multiple replicas:
         ```bash
         kubectl scale deployment nginx-deployment --replicas=3
         ```
      - Update the Deployment with a new image version:
         ```bash
         kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
         ```
   - **Exposing Services:**
      - Expose a Deployment as a Service:
         ```bash
         kubectl expose deployment nginx-deployment --type=NodePort --port=80
         ```
      - Get the URL to access the Service:
         ```bash
         minikube service nginx-deployment --url
         ```
   - **Interacting with ConfigMaps and Secrets:**
      - Create a ConfigMap:
         ```bash
         kubectl create configmap my-config --from-literal=key1=value1
         ```
      - Create a Secret:
         ```bash
         kubectl create secret generic my-secret --from-literal=username=admin --from-literal=password=secret
         ```
      - View the details of a ConfigMap:
         ```bash
         kubectl describe configmap my-config
         ```
      - View the details of a Secret:
         ```bash
         kubectl describe secret my-secret
         ```

### 4. **Hands-On Exercises:**
   - **Exercise 1 - Running a Pod:**
      - Start a simple Pod running Nginx and access its logs:
         ```bash
         kubectl run nginx --image=nginx --restart=Never
         kubectl logs nginx
         ```
   - **Exercise 2 - Working with Deployments:**
      - Create a Deployment and scale it to multiple replicas:
         ```bash
         kubectl create deployment myapp --image=nginx
         kubectl scale deployment myapp --replicas=4
         ```
   - **Exercise 3 - Networking and Services:**
      - Expose a Deployment as a Service and access it via Minikube:
         ```bash
         kubectl expose deployment myapp --type=NodePort --port=80
         minikube service myapp --url
         ```
   - **Exercise 4 - ConfigMaps and Secrets:**
      - Create a ConfigMap and a Secret, and use them in a Pod definition:
         ```bash
         kubectl create configmap app-config --from-literal=app_name=myapp
         kubectl create secret generic app-secret --from-literal=db_password=supersecret
         ```
         Modify the Pod YAML to include the ConfigMap and Secret:
         ```yaml
         apiVersion: v1
         kind: Pod
         metadata:
           name: myapp-pod
         spec:
           containers:
           - name: myapp-container
             image: nginx
             env:
             - name: APP_NAME
               valueFrom:
                 configMapKeyRef:
                   name: app-config
                   key: app_name
             - name: DB_PASSWORD
               valueFrom:
                 secretKeyRef:
                   name: app-secret
                   key: db_password
         ```
   - **Exercise 5 - Persistent Storage:**
      - Create a PersistentVolume and a PersistentVolumeClaim:
         ```yaml
         apiVersion: v1
         kind: PersistentVolume
         metadata:
           name: pv0001
         spec:
           capacity:
             storage: 1Gi
           accessModes:
             - ReadWriteOnce
           hostPath:
             path: "/mnt/data"
         ```
         ```yaml
         apiVersion: v1
         kind: PersistentVolumeClaim
         metadata:
           name: pvc0001
         spec:
           accessModes:
             - ReadWriteOnce
           resources:
             requests:
               storage: 1Gi
         ```
         Create a Pod that uses this storage:
         ```yaml
         apiVersion: v1
         kind: Pod
         metadata:
           name: myapp-pod
         spec:
           containers:
           - name: myapp-container
             image: nginx
             volumeMounts:
             - mountPath: "/usr/share/nginx/html"
               name: my-storage
           volumes:
           - name: my-storage
             persistentVolumeClaim:
               claimName: pvc0001
         ```
   - **Exercise 6 - Kubernetes Dashboard:**
      - Access the Kubernetes dashboard using Minikube:
         ```bash
         minikube dashboard
         ```
      - Explore different resources like Pods, Deployments, and Services from the dashboard.
   - **Exercise 7 - Helm Charts (Optional Advanced):**
      - Install Helm, a package manager for Kubernetes:
         ```bash
         curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
         ```
      - Install a sample Helm chart (e.g., WordPress):
         ```bash
         helm repo add bitnami https://charts.bitnami.com/bitnami
         helm install my-wordpress bitnami/wordpress
         ```

### 5. **Advanced Topics and Additional Exercises:**
   - **Exercise 8 - Rolling Updates and Rollbacks:**
      - Perform a rolling update on a Deployment:
         ```bash
         kubectl set image deployment/myapp-deployment myapp=nginx:1.16.1
         ```
      - Roll back to a previous version:
         ```bash
         kubectl rollout undo deployment/myapp-deployment
         ```
   - **Exercise 9 - Autoscaling:**
      - Enable autoscaling for a Deployment based on CPU usage:
         ```bash
         kubectl autoscale deployment myapp --min=1 --max=10 --cpu-percent=80
         ```
   - **Exercise 10 - Ingress Controllers:**
      - Install an Ingress controller (e.g., NGINX Ingress) and expose a service:
         ```bash
         kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
         ```
         Define an Ingress resource:
         ```yaml
         apiVersion: networking.k8s.io/v1
         kind: Ingress


         metadata:
           name: example-ingress
         spec:
           rules:
           - host: myapp.local
             http:
               paths:
               - path: /
                 pathType: Prefix
                 backend:
                   service:
                     name: myapp-service
                     port:
                       number: 80
         ```
   - **Exercise 11 - Monitoring with Prometheus and Grafana:**
      - Deploy Prometheus and Grafana for monitoring the Kubernetes cluster using Helm:
         ```bash
         helm install prometheus prometheus-community/kube-prometheus-stack
         ```
      - Access Grafana and visualize metrics from the Kubernetes cluster.
   - **Exercise 12 - Kubernetes Security Best Practices:**
      - Implement Role-Based Access Control (RBAC) in the cluster:
         ```yaml
         apiVersion: v1
         kind: ServiceAccount
         metadata:
           name: myapp-sa
         ```
         ```yaml
         apiVersion: rbac.authorization.k8s.io/v1
         kind: Role
         metadata:
           namespace: default
           name: pod-reader
         rules:
         - apiGroups: [""]
           resources: ["pods"]
           verbs: ["get", "watch", "list"]
         ```
         ```yaml
         apiVersion: rbac.authorization.k8s.io/v1
         kind: RoleBinding
         metadata:
           name: read-pods
           namespace: default
         subjects:
         - kind: ServiceAccount
           name: myapp-sa
           namespace: default
         roleRef:
           kind: Role
           name: pod-reader
           apiGroup: rbac.authorization.k8s.io
         ```
   - **Exercise 13 - Kubernetes Networking:**
      - Explore Kubernetes network policies to control the traffic between Pods:
         ```yaml
         apiVersion: networking.k8s.io/v1
         kind: NetworkPolicy
         metadata:
           name: allow-nginx
         spec:
           podSelector:
             matchLabels:
               run: nginx
           policyTypes:
           - Ingress
           ingress:
           - from:
             - podSelector:
                 matchLabels:
                   run: nginx
   - **Exercise 14 - Multi-Cluster Kubernetes:**
      - Explore how to manage multi-cluster Kubernetes environments using tools like `kubefed` or service meshes like Istio.

### 6. **Additional Resources:**
   - Online courses, tutorials, and documentation links for further learning:
     - Kubernetes Documentation: [https://kubernetes.io/docs/](https://kubernetes.io/docs/)
     - Minikube Documentation: [https://minikube.sigs.k8s.io/docs/](https://minikube.sigs.k8s.io/docs/)
   - Community forums and support channels for Kubernetes-related queries:
     - Kubernetes Slack: [https://slack.k8s.io/](https://slack.k8s.io/)
     - Stack Overflow: [https://stackoverflow.com/questions/tagged/kubernetes](https://stackoverflow.com/questions/tagged/kubernetes)

### 7. **Conclusion:**
   - Reflect on your Kubernetes learning journey with Minikube. Consider the challenges you faced, the concepts you grasped, and the skills you acquired.
   - Identify areas for further exploration, such as advanced Kubernetes topics, cloud-native architectures, or Kubernetes certifications like CKA (Certified Kubernetes Administrator).

This documentation serves as a comprehensive guide to learning Kubernetes using Minikube, with explanations, commands, and hands-on exercises. Adjust the content as needed based on your specific learning goals. Happy container orchestration!
