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
<img width="998" height="529" alt="Screenshot from 2025-08-23 00-36-15" src="https://github.com/user-attachments/assets/353285ad-4865-4a32-bafc-13bd330e4196" />


### 1.2 Dockerfile  
File: `Dockerfile`  
```
FROM python:3.8-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```
<img width="854" height="113" alt="Screenshot from 2025-08-23 00-43-41" src="https://github.com/user-attachments/assets/7b0653dd-0cb6-4e3d-9996-deda351ddf8f" />

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
  <img width="849" height="147" alt="Screenshot from 2025-08-23 00-42-36" src="https://github.com/user-attachments/assets/50c95ca3-c814-4588-8bd0-082c3f792453" />

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
  <img width="883" height="228" alt="Screenshot from 2025-08-23 00-44-34" src="https://github.com/user-attachments/assets/c93c7ebc-8f6d-4bab-a580-1652a3d76b5a" />

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
  <img width="1035" height="405" alt="Screenshot from 2025-08-23 00-45-02" src="https://github.com/user-attachments/assets/e5cf912a-c6cc-4804-9f88-ea617cb6c985" />


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
<img width="1035" height="405" alt="Screenshot from 2025-08-23 00-45-02" src="https://github.com/user-attachments/assets/9723ed6c-285e-48b5-814c-dbb9ccac12c2" />

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
<img width="983" height="545" alt="Screenshot from 2025-08-23 00-47-51" src="https://github.com/user-attachments/assets/f5233fc4-07c8-4c44-8b06-3af7ed3f2f27" />


### 2.2 Dockerfile  
File: `Dockerfile`  
```
FROM python:3.8-slim
WORKDIR /app
COPY bouncer.py .
CMD ["python", "bouncer.py"]
```
<img width="679" height="119" alt="Screenshot from 2025-08-23 00-48-17" src="https://github.com/user-attachments/assets/e46742d5-ed37-4bb0-9fb1-8b1f63c0c4c3" />


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
  <img width="735" height="136" alt="Screenshot from 2025-08-23 00-48-58" src="https://github.com/user-attachments/assets/e77f6bc7-3f29-4e60-bb14-75a876b4788c" />


- **values.yaml**  
  ```yaml
  image:
    repository: bouncer-xapp
    tag: latest
    pullPolicy: IfNotPresent

  replicaCount: 1
  ```
  <img width="756" height="154" alt="Screenshot from 2025-08-23 00-49-19" src="https://github.com/user-attachments/assets/e0a9fe6e-6747-428b-b924-3480734f5f0b" />

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
  <img width="949" height="400" alt="Screenshot from 2025-08-23 00-49-54" src="https://github.com/user-attachments/assets/ccab6774-22f5-465a-934c-e4762ff7ec82" />


### 2.4 Build & Deploy  
```bash
eval "$(minikube docker-env)"
docker build -t bouncer-xapp:latest .
eval "$(minikube docker-env -u)"

helm install bouncer-xapp helm/bouncer-xapp -n ricplt

kubectl get pods -n ricplt | grep bouncer-xapp
kubectl logs deployment/bouncer-xapp -n ricplt
```
<img width="1412" height="658" alt="Screenshot from 2025-08-23 00-51-34" src="https://github.com/user-attachments/assets/3091c12e-3107-40e7-a57f-563ea16a2042" />


***

## Summary  
- Two xApps were created and deployed, even though only one was requested.  
- Both demonstrate the same lifecycle: write minimal Python code, containerize with Docker, author a Helm chart, and deploy on Minikube under namespace `ricplt`.  
- Each xApp reached a **Running** state and produced the expected logs, confirming successful end-to-end deployment.


