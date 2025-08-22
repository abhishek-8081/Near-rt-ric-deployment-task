# Work Progress

## 1. Environment Setup

- Verified Docker installation  
  ```bash
  docker --version
  docker run hello-world
  ```
- Cleaned up and started Minikube with Docker driver  
  ```bash
  minikube delete
  minikube start --driver=docker --memory=3072 --cpus=4 --disk-size=20g
  kubectl get nodes
  ```
<!-- Add SS for Docker & Minikube setup -->

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
    --docker-username=<your-username> \
    --docker-password=<your-password> \
    --docker-email=<your-email> \
    -n ricplt
  ```
<!-- Add SS for nearrtric pods status -->

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

