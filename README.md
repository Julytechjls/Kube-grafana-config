                                                    Monitoring Project with Grafana and Prometheus in Kubernetes
                                      https://grafana.com/grafana/dashboards/315-kubernetes-cluster-monitoring-via-prometheus/

This project implements a monitoring system using Grafana and Prometheus deployed on a Kubernetes cluster. 
Below are the configuration files used and the steps required to deploy the services using Minikube:


Grafana:

grafana-configmap.yaml:      Defines a ConfigMap for Grafana, including configurations such as dashboards and other application configurations.
grafana-deployment.yaml:     Describes the Grafana Deployment, specifying the container image, number of replicas, and other pod settings.
grafana-hpa.yaml:            Configures Horizontal Pod Autoscaler (HPA) for Grafana, allowing Kubernetes to automatically adjust the number of pod replicas based on load.
grafana-pv.yaml:             Defines a PersistentVolume (PV) for Grafana, providing persistent storage.
grafana-pvc.yaml:            Describes a PersistentVolumeClaim (PVC) for Grafana, which requests storage defined by the PV.
grafana-service.yaml:        Configures the Grafana Service, exposing the Deployment through an IP address within the cluster.
grafana-datasource.yaml:     Contains the configuration for the Grafana data source, specifying Prometheus as the data source.


Prometheus:

prometheus.yaml:             Prometheus main configuration, specifying scraping rules and other settings.
prometheus-configmap.yaml:   Defines a ConfigMap for Prometheus, which includes additional application configurations.
prometheus-deployment.yaml:  Describes the Prometheus Deployment, specifying the container image, number of replicas, and other pod configurations.
prometheus-service.yaml:     Configures the Prometheus Service, exposing the Deployment through an IP address within the cluster.


Deployment to Kubernetes

Steps:
1.  Create a file called "grafana-configmap.yaml" with the following content:
   
apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  selector:
    app: prometheus
  ports:
  - protocol: TCP
    port: 9090
    targetPort: 9090
  type: LoadBalancer


2.  Create a file called "grafana-deployment.yaml" with the following content:
   
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment  # Name of the deployment, modify as necessary
spec:
  replicas: 1  # Number of pod replicas, adjust according to requirements
  selector:
    matchLabels:
      app: grafana-app  # Label to select the pods, must match the labels in template.metadata.labels
  template:
    metadata:
      labels:
        app: grafana-app  # Label for the pod, must match the selector
    spec:
      containers:
      - name: my-container  # Container name, modify as needed
        image: grafana/grafana:latest
        ports: 
        - containerPort: 3000  # Port in the container that the application exposes, adjust as necessary
        env:
        - name: GF_SECURITY_ADMIN_USER
          valueFrom:
            secretKeyRef:
              name: admin
              key: admin
        - name: GF_SECURITY_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: grafana-admin
              key: admin-password
              

3.  Create a file called "grafana-hpa.yaml" with the following content:
   
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: grafana-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: grafana
  minReplicas: 1
  maxReplicas: 3
  targetCPUUtilizationPercentage: 80
  

4.  Create a file called "grafana-pv.yaml" with the following content:

apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/grafana"
    

5.  Create a file called "grafana-pvc.yaml" with the following content:
   
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: "grafana-pv"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-pv
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /data/pv0001/
    

6.  Create a file called "grafana-service.yaml" with the following content:

apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  selector:
    app: grafana-app
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
  type: LoadBalancer


7.  Create a file called "grafana-datasource.yaml" with the following content:

apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasource
data:
  datasources.yaml: |
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        access: proxy
        url: http://prometheus:9090
        

8.  Create a file called "prometheus.yaml" with the following content:

apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s

    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'node-exporter'
        static_configs:
          - targets: ['node-exporter:9100']
          

9.  Create a file called "prometheus-configmap.yaml" with the following content:

apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |-
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'node-exporter'
        static_configs:
          - targets: ['node-exporter:9100']
          

10.  Create a file called "prometheus-deployment.yaml" with the following content:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config-volume
          mountPath: /etc/prometheus
      volumes:
      - name: config-volume
        configMap:
          name: prometheus-config


11.  Create a file called "prometheus-service.yaml" with the following content:

apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  selector:
    app: prometheus
  ports:
  - protocol: TCP
    port: 9090
    targetPort: 9090
  type: LoadBalancer

Add up all the deployment with the following command:
kubectl apply -f (deployment name)

Access Prometheus

To access the Prometheus service, use the following command to get the list of services and then open Prometheus in your browser:

kubectl get services
minikube service prometheus-service

Then open new browser with: localhost:8080

Conclusion

By following these steps, you will have successfully deployed and configured a monitoring system using Grafana and Prometheus on a Kubernetes cluster. 
Now you can start monitoring your applications and services, obtaining valuable information about their performance and status.

Additional Resources

https://prometheus.io/docs/
https://grafana.com/docs/
https://kubernetes.io/docs/
https://grafana.com/docs/grafana-cloud/
https://grafana.com/docs/grafana-cloud/developer-resources/infrastructure-as-code/grafana-operator/
https://grafana.com/docs/grafana/latest/introduction/grafana-cloud/


