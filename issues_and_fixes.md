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
  <img width="1431" height="98" alt="Screenshot from 2025-08-22 22-53-56" src="https://github.com/user-attachments/assets/78055f27-e931-4f7f-9d2b-1c168e7e11b6" />

  
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
<img width="1064" height="97" alt="Screenshot from 2025-08-22 22-59-16" src="https://github.com/user-attachments/assets/0a979c59-ca5b-48b9-a986-91c77374a3fa" />


**Fix:** Use correct chart path under `new-installer`.

```bash
helm install nearrtric ~/ric-dep/new-installer/helm/charts/nearrtric -n ricplt --create-namespace
```
<img width="1235" height="127" alt="Screenshot from 2025-08-22 23-02-55" src="https://github.com/user-attachments/assets/66735636-e268-47c9-9f0d-ffa09a185e4b" />


### 2. Namespace Not Found

**Issue:** Release required `ricxapp` namespace that did not exist.

```bash
# Error: namespaces "ricxapp" not found
```
<img width="1230" height="41" alt="Screenshot from 2025-08-22 23-04-45" src="https://github.com/user-attachments/assets/5d11ee21-2cab-4e69-af6a-ce17cb895a24" />

**Fix:** Create the missing namespace.

```bash
kubectl create namespace ricxapp
```
<img width="1230" height="41" alt="Screenshot from 2025-08-22 23-04-45" src="https://github.com/user-attachments/assets/12de7169-9ac8-4f1f-9a93-f3cf4125c6dd" />


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
  --docker-username=abhi \
  --docker-password=xyz\
  --docker-email=abhishekrajputji2004@gmail.com \
  -n ricplt
```
<img width="869" height="114" alt="Screenshot from 2025-08-22 21-59-27" src="https://github.com/user-attachments/assets/69b62deb-4fad-462e-8ec3-471ccd6adffa" />

Attach this secret to the service account or reference it in the chart’s `imagePullSecrets`.

### 5. ConfigMap Ownership & Stale Resources

**Issue:** `xapp-onboarder` install kept failing due to leftover ConfigMaps and Helm secrets from previous attempts.

```bash
helm install xapp-onboarder ./xapp-onboarder -n ricplt
# Error: configmaps "configmap-ricplt-xapp-onboarder-env" already exists
```
<img width="1186" height="37" alt="Screenshot from 2025-08-22 23-16-16" src="https://github.com/user-attachments/assets/6292bf0d-322e-4d22-af68-53f47af22ecd" />


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
<img width="1406" height="112" alt="Screenshot from 2025-08-22 23-19-36" src="https://github.com/user-attachments/assets/6bd61645-474c-4a86-b7fc-6ae75cf316c8" />

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

