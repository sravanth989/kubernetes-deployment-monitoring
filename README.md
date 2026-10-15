# Kubernetes Deployment, Autoscaling & Monitoring

A hands-on DevOps lab using **Kubernetes (Minikube), Helm, Prometheus, and Grafana** to deploy a sample application, configure Horizontal Pod Autoscaling (HPA), and observe cluster metrics.

## Project Overview

- Started a local Kubernetes cluster with Minikube.
- Deployed a sample NGINX application using Kubernetes manifests.
- Exposed the application through a Kubernetes Service.
- Configured HPA using CPU utilization and verified scale-up/scale-down during the lab.
- Installed the Prometheus and Grafana monitoring stack with Helm.
- Configured Prometheus as a Grafana data source and explored dashboards/alerting.

> **Note:** The YAML examples below are clean, reusable manifests prepared to document the project. Review and test them in your own cluster before presenting them as the exact files used during the original lab.

## Architecture

`User → Kubernetes Service → NGINX Deployment/Pods → HPA`

`Kubernetes Metrics Server → HPA metrics`

`Prometheus → metrics collection → Grafana dashboards/alerts`

## Tools Used

Kubernetes · Minikube · kubectl · Metrics Server · Helm · Prometheus · Grafana · YAML

## 1. Start Minikube

```bash
minikube start
kubectl get nodes
kubectl get pods -A
minikube addons enable metrics-server
```

## 2. Deploy the Application

Save the following as `deployment.yaml` when testing locally:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 500m
              memory: 128Mi
```

Apply and verify:

```bash
kubectl apply -f deployment.yaml
kubectl get deployments,pods
```

## 3. Expose the Application

Save as `service.yaml` when testing locally:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

```bash
kubectl apply -f service.yaml
kubectl get service nginx-service
minikube service nginx-service
```

## 4. Configure Horizontal Pod Autoscaling

Save as `hpa.yaml` when testing locally:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
kubectl describe hpa nginx-hpa
kubectl get pods -w
```

For HPA to work, the Metrics Server must be available and containers must specify CPU requests. During the lab, HPA scaling was observed from 2 to 5 replicas and back down after load decreased.

## 5. Install Monitoring with Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
kubectl get pods -n monitoring
```

Port-forward Grafana locally:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Open `http://localhost:3000`. Retrieve the chart-generated admin password using the chart's documented secret instructions; do not commit credentials or passwords to GitHub.

The Prometheus/Grafana lab included configuring Prometheus as Grafana's data source, viewing Kubernetes dashboards, and exploring alert rules.

## 6. Useful Verification Commands

```bash
kubectl get nodes
kubectl get deployments
kubectl get pods -o wide
kubectl get services
kubectl get hpa
kubectl top pods
helm list -A
kubectl get pods -n monitoring
```

## 7. Cleanup

```bash
helm uninstall monitoring -n monitoring
kubectl delete -f hpa.yaml
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
minikube stop
```

## Skills Demonstrated

- Kubernetes application deployment and service exposure
- Resource requests and limits
- Horizontal Pod Autoscaling concepts and verification
- Helm-based monitoring stack installation
- Prometheus/Grafana dashboard and alerting exploration
- Command-line troubleshooting and validation

## Project Summary

Implemented a local Kubernetes deployment and monitoring lab with Minikube. Configured HPA to scale application replicas based on CPU utilization, then installed Prometheus and Grafana with Helm to inspect cluster metrics and dashboards.
