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
- Built and tagged Docker image locally  
  ```bash
  docker build -t kpi-monitor-xapp:latest .
  ```
- Installed via Helm into `ricplt` namespace  
  ```bash
  helm install kpi-monitor helm/kpi-monitor-xapp -n ricplt --create-namespace
  kubectl get pods -n ricplt
  ```
<!-- Add SS for kpi-monitor-xapp deployment -->

## 4. Developed & Deployed Bouncer xApp

- Scaffolded application and Helm chart  
  ```bash
  mkdir ~/bouncer-xapp
  # Created bouncer.py and Dockerfile
  mkdir -p helm/bouncer-xapp/templates
  # Created Chart.yaml, values.yaml, deployment.yaml
  ```
- Built and tagged Docker image locally  
  ```bash
  docker build -t bouncer-xapp:latest .
  ```
- Installed via Helm into `ricplt` namespace  
  ```bash
  helm install bouncer-xapp helm/bouncer-xapp -n ricplt
  kubectl get pods -n ricplt
  ```
- Resolved image pull issues by using Minikube Docker environment  
  ```bash
  eval "$(minikube docker-env)"
  docker build -t bouncer-xapp:latest .
  eval "$(minikube docker-env -u)"
  helm upgrade bouncer-xapp ~/bouncer-xapp/helm/bouncer-xapp -n ricplt --reuse-values
  kubectl get pods -n ricplt
  ```
<!-- Add SS for bouncer-xapp logs and status -->

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
- Linted and updated dependencies for `xapp-onboarder` chart  
  ```bash
  helm lint ./xapp-onboarder
  helm dependency update ./xapp-onboarder
  helm repo update
  ```
- Attempted fresh install of `xapp-onboarder`  
  ```bash
  helm install xapp-onboarder ./xapp-onboarder -n ricplt
  ```
  *Encountered reuse/ownership errors; cleanup in progress.*

<!-- Add SS for cleanup commands and lint output -->

***

*Progress snapshots and additional details to be added.*


