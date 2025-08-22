# Work Progress

## 1. Environment Setup

- Verified Docker installation  
  ```bash
  docker --version
  docker run hello-world
  ```
  <img width="880" height="550" alt="Screenshot from 2025-08-22 23-39-24" src="https://github.com/user-attachments/assets/49704c66-0f9e-4a92-9937-72a9bb03d8a3" />

- Cleaned up and started Minikube with Docker driver  
  ```bash
  minikube delete
  minikube start --driver=docker --memory=3072 --cpus=4 --disk-size=20g
  kubectl get nodes
  ```
  <img width="1417" height="527" alt="Screenshot from 2025-08-22 23-40-49" src="https://github.com/user-attachments/assets/0d94bf2c-2396-4273-991a-8e91575a40bb" />

## 2. Deployed Near-RT RIC Platform Chart

- Installed `nearrtric` from `new-installer` path  
  ```bash
  kubectl create namespace ricxapp
  helm uninstall nearrtric -n ricplt || true
  helm install nearrtric ~/ric-dep/new-installer/helm/charts/nearrtric -n ricplt --create-namespace
  kubectl get pods -n ricplt
  ```
- Troubleshot image pull errors by creating registry secret  
  ```bash
  kubectl create secret docker-registry nexus3-secret \
    --docker-server=nexus3.o-ran-sc.org:10002 \
    --docker-username=abhi \
    --docker-password=xyz \
    --docker-email=abhishekrajputji2004@gmail.com \
    -n ricplt
  ```
  <img width="869" height="130" alt="Screenshot from 2025-08-22 23-45-04" src="https://github.com/user-attachments/assets/b106c53c-2fe9-4768-affa-87e0b908144f" />
  
  <img width="940" height="209" alt="Screenshot from 2025-08-22 23-48-56" src="https://github.com/user-attachments/assets/9e6ef7ce-d7d1-44af-a127-9b6be7e40f0c" />

## 3. Developed & Deployed KPI Monitor xApp

- Scaffolded application and Helm chart  
  ```bash
  mkdir ~/kpi-monitor-xapp
  # Created app.py and Dockerfile
  mkdir -p helm/kpi-monitor-xapp/templates
  # Created Chart.yaml, values.yaml, deployment.yaml
  ```
  <img width="877" height="603" alt="Screenshot from 2025-08-22 23-56-55" src="https://github.com/user-attachments/assets/03da1b6b-0a66-44d2-8f36-f1ee579a88d6" />

- Built and tagged Docker image locally  
  ```bash
  docker build -t kpi-monitor-xapp:latest .
  ```
  <img width="853" height="486" alt="Screenshot from 2025-08-22 23-58-03" src="https://github.com/user-attachments/assets/48b0fdc4-e6d8-4451-b24f-5134c6a39d3e" />

- Installed via Helm into `ricplt` namespace  
  ```bash
  helm install kpi-monitor helm/kpi-monitor-xapp -n ricplt --create-namespace
  kubectl get pods -n ricplt
  ```
  <img width="853" height="486" alt="Screenshot from 2025-08-22 23-58-03" src="https://github.com/user-attachments/assets/0d852b99-e4fd-4c48-9f19-8760f0842dc3" />

## 4. Developed & Deployed Bouncer xApp

- Scaffolded application and Helm chart  
  ```bash
  mkdir ~/bouncer-xapp
  # Created bouncer.py and Dockerfile
  mkdir -p helm/bouncer-xapp/templates
  # Created Chart.yaml, values.yaml, deployment.yaml
  ```
  <img width="983" height="817" alt="Screenshot from 2025-08-23 00-01-52" src="https://github.com/user-attachments/assets/495f9812-adc3-471d-a2cb-30869841ad66" />

- Built and tagged Docker image locally  
  ```bash
  docker build -t bouncer-xapp:latest .
  ```
  <img width="846" height="346" alt="Screenshot from 2025-08-23 00-03-02" src="https://github.com/user-attachments/assets/58fe680a-86a4-4ccd-9664-24c013ed16be" />

- Installed via Helm into `ricplt` namespace  
  ```bash
  helm install bouncer-xapp helm/bouncer-xapp -n ricplt
  kubectl get pods -n ricplt
  ```
  <img width="1058" height="316" alt="Screenshot from 2025-08-23 00-03-38" src="https://github.com/user-attachments/assets/3e1f2164-47e1-410a-a06b-eb3e804571ad" />

- Resolved image pull issues by using Minikube Docker environment  
  ```bash
  eval "$(minikube docker-env)"
  docker build -t bouncer-xapp:latest .
  eval "$(minikube docker-env -u)"
  helm upgrade bouncer-xapp ~/bouncer-xapp/helm/bouncer-xapp -n ricplt --reuse-values
  kubectl get pods -n ricplt
  ```
  <img width="1351" height="498" alt="Screenshot from 2025-08-23 00-05-19" src="https://github.com/user-attachments/assets/93696cf8-7791-4c66-a051-27057fc6c458" />

## 5. Chart Cleanup and Retry

- Removed stale ConfigMaps and Helm secrets for `xapp-onboarder`  
  ```bash
  kubectl delete configmap \
    configmap-ricplt-xapp-onboarder-env \
    configmap-ricplt-xapp-onboarder-chartmuseum-env \
    -n ricplt --ignore-not-found
  kubectl delete secret sh.helm.release.v1.xapp-onboarder.v1 -n ricplt --ignore-not-found
  helm uninstall xapp-onboarder -n ricplt || true
  ```
  <img width="1300" height="331" alt="Screenshot from 2025-08-23 00-23-13" src="https://github.com/user-attachments/assets/76c5c736-57e7-4984-a33b-d70c08be96de" />

- Linted and updated dependencies for `xapp-onboarder` chart  
  ```bash
  helm lint ./xapp-onboarder
  helm dependency update ./xapp-onboarder
  helm repo update
  ```
  <img width="795" height="112" alt="Screenshot from 2025-08-23 00-25-52" src="https://github.com/user-attachments/assets/47f35932-2a64-4030-9835-00e6ebd3ece5" />

- Attempted fresh install of `xapp-onboarder`  
  ```bash
  helm install xapp-onboarder ./xapp-onboarder -n ricplt
  ```
  <img width="1785" height="188" alt="Screenshot from 2025-08-23 00-29-00" src="https://github.com/user-attachments/assets/22224c41-e9ef-42b6-9bc5-23076f29a684" />

  *Encountered reuse/ownership errors; cleanup in progress.*

***


