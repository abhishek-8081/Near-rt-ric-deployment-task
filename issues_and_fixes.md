# Issues and Fixes

## OS Compatibility

- **Ubuntu 20.04 (amd64)**
  - Docker version 26.1.3 (build 26.1.3-0ubuntu1~20.04.1)
  - Minikube v1.36.0 on Ubuntu 20.04 using Docker driver

> **Note:** Ensure Docker and Minikube are installed and supported on Ubuntu 20.04.

<!-- Add OS compatibility screenshot here -->

## Space Issue

- **Minikube memory allocation warning:**
  ```bash
  minikube start --driver=docker --memory=3072 --cpus=4 --disk-size=20g
  # 🧯  The requested memory allocation of 3072MiB does not leave room for system overhead...
  ```
> **Suggestion:** Adjust `--memory` to leave system overhead.

<!-- Add space issue screenshot here -->

## Issues and Fixes

### 1. Helm Chart Path Not Found

**Issue:** `nearrtric` chart directory was misreferenced, causing “path not found.”

```bash
cd ~/ric-dep/helm
helm install nearrtric ./nearrtric -n ricplt --create-namespace
# Error: path "./nearrtric" not found
```

**Fix:** Use correct chart path under `new-installer`.

```bash
helm install nearrtric ~/ric-dep/new-installer/helm/charts/nearrtric -n ricplt --create-namespace
```

### 2. Namespace Not Found

**Issue:** Release required `ricxapp` namespace that did not exist.

```bash
# Error: namespaces "ricxapp" not found
```

**Fix:** Create the missing namespace.

```bash
kubectl create namespace ricxapp
```

### 3. Release Name Reuse

**Issue:** Helm refused to reuse an existing release name after a failed install.

```bash
helm install nearrtric ... 
# Error: cannot re-use a name that is still in use
```

**Fix:** Uninstall the existing release before re-installation.

```bash
helm uninstall nearrtric -n ricplt
helm install nearrtric ... 
```

### 4. ImagePullBackOff Errors

**Issue:** E2 Manager and RT Manager pods failed to pull images due to missing manifests and lack of registry credentials.

```bash
kubectl describe pod deployment-ricplt-e2mgr-... -n ricplt
# Reason: ImagePullBackOff (manifest unknown)
```

**Fix:** Create a Kubernetes Docker registry secret for the Nexus repository.

```bash
kubectl create secret docker-registry nexus3-secret \
  --docker-server=nexus3.o-ran-sc.org:10002 \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email> \
  -n ricplt
```
Attach this secret to the service account or reference it in the chart’s `imagePullSecrets`.

### 5. ConfigMap Ownership & Stale Resources

**Issue:** `xapp-onboarder` install kept failing due to leftover ConfigMaps and Helm secrets from previous attempts.

```bash
helm install xapp-onboarder ./xapp-onboarder -n ricplt
# Error: configmaps "configmap-ricplt-xapp-onboarder-env" already exists
```

**Fix:** Remove stale ConfigMaps, Helm secrets, and any failed release metadata.

```bash
kubectl delete configmap \
  configmap-ricplt-xapp-onboarder-env \
  configmap-ricplt-xapp-onboarder-chartmuseum-env \
  -n ricplt --ignore-not-found

kubectl delete secret \
  sh.helm.release.v1.xapp-onboarder.v1 \
  -n ricplt --ignore-not-found

helm uninstall xapp-onboarder -n ricplt || true
```

### 6. Helm Dependency Errors

**Issue:** Local chart dependency `ric-common` couldn’t be fetched from the local repo, blocking installation.

```bash
helm dependency update ./xapp-onboarder
# Error: ric-common chart not found in repo http://localhost:18080/
```

**Fix:** Ensure the local chart repository is running or update dependencies from remote.

```bash
helm repo update
helm dependency update ./xapp-onboarder
helm install xapp-onboarder ./xapp-onboarder -n ricplt
```

***

*(Leave space for adding relevant screenshots)*
