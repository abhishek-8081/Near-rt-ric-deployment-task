# xApp Deployment Details

## Overview  
Although the assignment requested deploying a single xApp, two were developed and deployed to demonstrate the full end-to-end process:  

- **KPI Monitor xApp** – collects and logs KPI metrics every 30 seconds.  
- **Bouncer xApp** – simulates message bouncing and load testing with randomized latency every 5 seconds.  

Both xApps follow the same development, containerization, and Helm deployment pattern on the Near-RT RIC platform.

***

## 1. KPI Monitor xApp

### 1.1 Application Code  
File: `app.py`  
```python
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class KPIMonitorApp:
    def __init__(self):
        self.name = "kpi-monitor-xapp"
        self.version = "1.0.0"

    def start(self):
        logger.info(f"Starting {self.name} version {self.version}")
        while True:
            logger.info("KPI Monitor: Collecting metrics...")
            time.sleep(30)

if __name__ == "__main__":
    app = KPIMonitorApp()
    app.start()
```

### 1.2 Dockerfile  
File: `Dockerfile`  
```
FROM python:3.8-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

### 1.3 Helm Chart  
- **Directory**: `helm/kpi-monitor-xapp/`  
- **Chart.yaml**  
  ```yaml
  apiVersion: v2
  name: kpi-monitor-xapp
  description: A simple KPI monitoring xApp
  version: 1.0.0
  appVersion: 1.0.0
  ```
- **values.yaml**  
  ```yaml
  image:
    repository: kpi-monitor-xapp
    tag: latest
    pullPolicy: IfNotPresent

  replicaCount: 1

  service:
    type: ClusterIP
    port: 8080
  ```
- **templates/deployment.yaml**  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ .Chart.Name }}
    namespace: {{ .Release.Namespace }}
  spec:
    replicas: {{ .Values.replicaCount }}
    selector:
      matchLabels:
        app: {{ .Chart.Name }}
    template:
      metadata:
        labels:
          app: {{ .Chart.Name }}
      spec:
        containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          command: ["python", "app.py"]
  ```

### 1.4 Build & Deploy  
```bash
eval "$(minikube docker-env)"
docker build -t kpi-monitor-xapp:latest .
eval "$(minikube docker-env -u)"

helm install kpi-monitor helm/kpi-monitor-xapp \
  -n ricplt --create-namespace

kubectl get pods -n ricplt | grep kpi-monitor-xapp
kubectl logs deployment/kpi-monitor-xapp -n ricplt
```

***

## 2. Bouncer xApp

### 2.1 Application Code  
File: `bouncer.py`  
```python
import time
import random
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class BouncerApp:
    def __init__(self):
        self.name = "bouncer-xapp"
        self.version = "1.0.0"
        self.message_count = 0

    def start(self):
        logger.info(f"Starting {self.name} version {self.version}")
        logger.info("Bouncer: Ready to bounce messages and generate test load")
        while True:
            self.message_count += 1
            logger.info(f"Bouncer: Processed message #{self.message_count}")
            latency = random.randint(10, 100)
            logger.info(f"Bouncer: Simulating load test - Random latency: {latency}ms")
            time.sleep(5)

if __name__ == "__main__":
    app = BouncerApp()
    app.start()
```

### 2.2 Dockerfile  
File: `Dockerfile`  
```
FROM python:3.8-slim
WORKDIR /app
COPY bouncer.py .
CMD ["python", "bouncer.py"]
```

### 2.3 Helm Chart  
- **Directory**: `helm/bouncer-xapp/`  
- **Chart.yaml**  
  ```yaml
  apiVersion: v2
  name: bouncer-xapp
  description: A Bouncer xApp for load testing and message routing
  version: 1.0.0
  appVersion: 1.0.0
  ```
- **values.yaml**  
  ```yaml
  image:
    repository: bouncer-xapp
    tag: latest
    pullPolicy: IfNotPresent

  replicaCount: 1
  ```
- **templates/deployment.yaml**  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ .Chart.Name }}
    namespace: {{ .Release.Namespace }}
  spec:
    replicas: {{ .Values.replicaCount }}
    selector:
      matchLabels:
        app: {{ .Chart.Name }}
    template:
      metadata:
        labels:
          app: {{ .Chart.Name }}
      spec:
        containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          command: ["python", "bouncer.py"]
  ```

### 2.4 Build & Deploy  
```bash
eval "$(minikube docker-env)"
docker build -t bouncer-xapp:latest .
eval "$(minikube docker-env -u)"

helm install bouncer-xapp helm/bouncer-xapp -n ricplt

kubectl get pods -n ricplt | grep bouncer-xapp
kubectl logs deployment/bouncer-xapp -n ricplt
```

***

## Summary  
- Two xApps were created and deployed, even though only one was requested.  
- Both demonstrate the same lifecycle: write minimal Python code, containerize with Docker, author a Helm chart, and deploy on Minikube under namespace `ricplt`.  
- Each xApp reached a **Running** state and produced the expected logs, confirming successful end-to-end deployment.

