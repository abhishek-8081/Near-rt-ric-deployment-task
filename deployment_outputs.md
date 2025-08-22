# Deployment Outputs

This document captures the complete deployment process, commands executed, and outputs observed during the Near-RT RIC platform deployment task. All outputs and status information documented here represent actual system states achieved during the deployment process.

---

## Environment Setup and Initial Configuration

### System Environment
- **Platform**: Ubuntu 20.04 on HP Laptop 15s VM
- **Kubernetes**: v1.33.1 via minikube
- **Docker**: v26.1.3
- **Container Runtime**: Docker with minikube driver
- **Resource Allocation**: 3GB RAM, 4 CPUs, 20GB disk

### Initial Setup Commands Executed

**Verify Docker installation**
```bash
docker --version
```

<img width="581" height="41" alt="Docker_version" src="https://github.com/user-attachments/assets/d2bc2148-24de-4d4e-bb97-5501ce4178fb" />

**Test Docker functionality**
```bash
docker run hello-world
```

<img width="939" height="512" alt="Docker Run Hello World" src="https://github.com/user-attachments/assets/98eaa9dc-3a18-4c0a-85b2-84eeb9c62f17" />

**Clean previous minikube cluster**
```bash
minikube delete
```

<img width="729" height="100" alt="Screenshot from 2025-08-22 16-18-35" src="https://github.com/user-attachments/assets/e5345374-89af-4967-8ed5-6ece4c989177" />

**Start minikube cluster with resource allocation**
```bash
minikube start --driver=docker --memory=3072 --cpus=4 --disk-size=20g
```

<img width="1540" height="359" alt="Screenshot from 2025-08-22 16-19-20" src="https://github.com/user-attachments/assets/8b4fcea5-29c9-45a9-9ce9-f4da54f9ebee" />

**Verify cluster nodes**
```bash
kubectl get nodes
```

<img width="870" height="57" alt="Screenshot from 2025-08-22 16-20-22" src="https://github.com/user-attachments/assets/a9497a3b-65f0-43b4-abbd-a603ab8174ef" />

---

## 1. Near-RT RIC Platform Deployment

### Commands Executed

**Navigate to helm directory**
```bash
cd ~/ric-dep/helm
```

<img width="974" height="54" alt="Screenshot from 2025-08-22 18-10-35" src="https://github.com/user-attachments/assets/d49878f6-4d2e-4570-8f59-59ea6389e149" />

**List available helm charts**
```bash
ls -l ~/ric-dep/new-installer/helm/charts/
```

<img width="819" height="100" alt="Screenshot from 2025-08-22 18-11-33" src="https://github.com/user-attachments/assets/5ab9584b-80bd-4d2e-85e6-d66b1def239b" />

**Create required namespace and deploy platform**
```bash
kubectl create namespace ricxapp
helm install nearrtric ~/ric-dep/new-installer/helm/charts/nearrtric -n ricplt --create-namespace
```

<img width="1235" height="239" alt="Screenshot from 2025-08-22 18-14-31" src="https://github.com/user-attachments/assets/c6244172-9478-4798-a105-232a074d687d" />

### Deployment Status Verification

**Monitor pod deployment status**
```bash
kubectl get pods -n ricplt
```

<img width="920" height="149" alt="Screenshot from 2025-08-22 18-20-49" src="https://github.com/user-attachments/assets/82da87e6-b85c-4713-af52-ecaee067e59f" />

**Debug E2 Manager pod issues**
```bash
kubectl describe pod deployment-ricplt-e2mgr-6dbf74545f-8w7pd -n ricplt
```

<img width="1595" height="812" alt="Screenshot from 2025-08-22 18-22-59" src="https://github.com/user-attachments/assets/bb912130-cec6-4854-99ed-c07f31e847ca" />

**Debug Routing Manager pod issues**
```bash
kubectl describe pod deployment-ricplt-rtmgr-677cfb9dd7-zgrqc -n ricplt
```

<img width="1564" height="774" alt="Screenshot from 2025-08-22 18-23-45" src="https://github.com/user-attachments/assets/bf7f77c4-fa2a-4e8a-9f06-e8913650a86f" />

### Platform Pods Status Output
```
NAME                                              READY   STATUS             RESTARTS      AGE
deployment-ricplt-appmgr-759b484cb4-srrq9         1/1     Running            0             16m
deployment-ricplt-e2mgr-6dbf74545f-8w7pd          0/1     ImagePullBackOff   0             16m
deployment-ricplt-e2term-alpha-6d6bbc684d-qdchl   0/1     CrashLoopBackOff   7 (62s ago)   16m
deployment-ricplt-rtmgr-677cfb9dd7-zgrqc          0/1     ImagePullBackOff   0             16m
deployment-ricplt-submgr-7f885656cb-mcgnv         1/1     Running            0             16m
statefulset-ricplt-dbaas-server-0                 1/1     Running            0             16m
```

<img width="956" height="169" alt="Screenshot from 2025-08-22 20-13-35" src="https://github.com/user-attachments/assets/c171cfb6-279f-4210-9c31-d0b395966b07" />

### Successfully Deployed Core Components
- ✅ **Application Manager** (deployment-ricplt-appmgr): Running
- ✅ **Subscription Manager** (deployment-ricplt-submgr): Running  
- ✅ **Database Service** (statefulset-ricplt-dbaas-server): Running
- ⚠️ **E2 Manager**: ImagePullBackOff (private registry access issue)
- ⚠️ **Routing Manager**: ImagePullBackOff (private registry access issue)
- ⚠️ **E2 Termination**: CrashLoopBackOff

---

## 2. xApp Deployments

### KPI Monitor xApp

#### Commands Executed

**Create xApp directory structure**
```bash
mkdir ~/kpi-monitor-xapp
cd ~/kpi-monitor-xapp
```

<img width="524" height="43" alt="Screenshot from 2025-08-22 18-29-07" src="https://github.com/user-attachments/assets/d17b2799-6927-43c9-a8db-dabf99aa128a" />

**Create Python application file**
```bash
cat > app.py << 'EOF'
import time
import json
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
EOF
```

<img width="524" height="43" alt="Screenshot from 2025-08-22 18-29-07" src="https://github.com/user-attachments/assets/c1211655-7928-4f2d-89b0-f50afb5267c9" />

**Create Docker configuration**
```bash
cat > Dockerfile << 'EOF'
FROM python:3.8-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
EOF
```

<img width="634" height="115" alt="Screenshot from 2025-08-22 18-31-15" src="https://github.com/user-attachments/assets/9fd839ec-643b-4c13-a993-bc2010b05fb5" />

**Create Helm chart structure**
```bash
mkdir -p helm/kpi-monitor-xapp/templates
cat > helm/kpi-monitor-xapp/chart.yaml<<'EOF'
```

<img width="946" height="146" alt="Screenshot from 2025-08-22 18-33-53" src="https://github.com/user-attachments/assets/9039cdd6-ea68-4fa8-ba71-7a4a3f180435" />

```bash
cat >helm/kpi-monitor-xapp/values.yaml<<'EOF'
cat >helm/kpi-monitor-xapp/templates/deployents.yaml<<'EOF'
```

<img width="946" height="146" alt="Screenshot from 2025-08-22 18-33-53" src="https://github.com/user-attachments/assets/01dc76e7-b1ad-4dbe-9073-72a0247b9d1d" />

**Build Docker image and deploy with Helm**
```bash
docker build -t kpi-monitor-xapp:latest .
helm install kpi-monitor helm/kpi-monitor-xapp -n ricplt --create-namespace
```

### Bouncer xApp

#### Commands Executed

**Create second xApp directory**
```bash
mkdir ~/bouncer-xapp
cd ~/bouncer-xapp
```

**Create Bouncer application file**
```bash
cat > bouncer.py << 'EOF'
import time
import json
import logging
import random

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
            logger.info(f"Bouncer: Simulating load test - Random latency: {random.randint(10,100)}ms")
            time.sleep(5)

if __name__ == "__main__":
    app = BouncerApp()
    app.start()
EOF
```

**Build and deploy Bouncer xApp**
```bash
docker build -t bouncer-xapp:latest .
helm install bouncer-xapp helm/bouncer-xapp -n ricplt
```

<img width="930" height="342" alt="Screenshot from 2025-08-22 20-16-53" src="https://github.com/user-attachments/assets/3606075e-268f-4178-9091-858c1396f504" />

### xApp Resolution Process

**Set minikube Docker environment**
```
eval "$(minikube docker-env)"
```
<img width="563" height="31" alt="Screenshot from 2025-08-23 01-45-21" src="https://github.com/user-attachments/assets/ab1eec39-2c5f-4ddb-8270-b3b376bd7c23" />

**Rebuild images within minikube context**
```bash
docker build -t kpi-monitor-xapp:latest .
docker build -t bouncer-xapp:latest .
```
<img width="966" height="711" alt="Screenshot from 2025-08-23 01-49-58" src="https://github.com/user-attachments/assets/28d37ead-9230-4243-8444-eca2fd176478" />

**Upgrade deployments with rebuilt images**
```bash
helm upgrade kpi-monitor ~/kpi-monitor-xapp/helm/kpi-monitor-xapp -n ricplt --reuse-values
helm upgrade bouncer-xapp ~/bouncer-xapp/helm/bouncer-xapp -n ricplt --reuse-values
```
<img width="964" height="260" alt="Screenshot from 2025-08-23 01-51-39" src="https://github.com/user-attachments/assets/32015a2f-0c01-4462-a117-25d02924afbb" />

### xApp Status Output
```
NAME                                              READY   STATUS             RESTARTS         AGE
bouncer-xapp-cc5b56588-tmp6w                      1/1     Running            0                7m6s
kpi-monitor-xapp-859bb4f59b-6w86j                 0/1     Error              1 (4s ago)       13m
```
<img width="747" height="161" alt="Screenshot from 2025-08-23 01-52-12" src="https://github.com/user-attachments/assets/4d1099b6-77fe-44a3-967a-53fae10f63c8" />

### Bouncer xApp Runtime Logs
```
INFO:__main__:Starting bouncer-xapp version 1.0.0
INFO:__main__:Bouncer: Ready to bounce messages and generate test load
INFO:__main__:Bouncer: Processed message #1
INFO:__main__:Bouncer: Simulating load test - Random latency: 37ms
INFO:__main__:Bouncer: Processed message #2
INFO:__main__:Bouncer: Simulating load test - Random latency: 16ms
...
```
<img width="747" height="161" alt="Screenshot from 2025-08-23 01-52-12" src="https://github.com/user-attachments/assets/7bbbe0a6-e236-423a-a23d-5c0fdb3070ca" />


---

## 3. xApp Onboarder Integration Attempts

### Commands Executed for xApp Onboarder

**Install xApp onboarder**
```bash
helm install xapp-onboarder ./xapp-onboarder -n ricplt
```

**Clean conflicting ConfigMaps**
```bash
kubectl delete configmap configmap-ricplt-xapp-onboarder-env -n ricplt
kubectl delete configmap configmap-ricplt-xapp-onboarder-chartmuseum-env -n ricplt
helm uninstall xapp-onboarder -n ricplt
```

**Update dependencies and repositories**
```bash
helm dependency update ./xapp-onboarder
helm repo update
```

### xApp Onboarder Status
- **Status**: Installation blocked by dependency issues
- **Issue**: Missing ric-common chart dependency from local repository
- **Error**: `ric-common chart not found in repo http://localhost:18080/`

---

## 4. E2 Simulator Integration

### Commands Executed

**Search for E2 simulator components**
```bash
find ~/ric-dep -name "*e2sim*" -o -name "*simulator*"
find ~/ric-dep -name "*e2*" -type d | grep -i sim
ls ~/ric-dep/*/
```

### E2 Simulator Status
- **Status**: No dedicated E2 simulator Helm chart found
- **Available Components**: E2 Manager and E2 Termination deployed as part of platform
- **Note**: E2 components present but experiencing image pull issues

---

## 5. Services and Network Status

### Services Verification Commands

**List all services in ricplt namespace**
```bash
kubectl get svc -n ricplt
```

**Get all resources in ricplt namespace**
```bash
kubectl get all -n ricplt
```

**List all Helm releases**
```bash
helm list -n ricplt
```

### Helm Releases Status
```
NAME           NAMESPACE REVISION STATUS   CHART               APP VERSION
nearrtric     ricplt   1       deployed nearrtric-0.1.0              
bouncer-xapp  ricplt   3       deployed bouncer-xapp-1.0.0           
kpi-monitor   ricplt   3       deployed kpi-monitor-xapp-1.0.0       
```

---

## 6. Issues Encountered and Resolutions

### Issue 1: Container Image Access
**Problem**: ImagePullBackOff for RIC platform components  
**Root Cause**: Private registry `nexus3.o-ran-sc.org:10002` requiring authentication  
**Error Messages**: 
```
Failed to pull image "nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-e2mgr:3.0.1": 
manifest for nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-e2mgr:3.0.1 not found: manifest unknown
```
**Status**: Partial resolution - core services running

### Issue 2: xApp Image Pull Issues
**Problem**: Custom xApp images not accessible from minikube  
**Solution Applied**:
```bash
eval "$(minikube docker-env)"
# Rebuilt images in minikube context
```
**Result**: ✅ Bouncer xApp successfully running

### Issue 3: Helm Chart Dependencies
**Problem**: xapp-onboarder installation failing due to missing dependencies  
**Commands Used**:
```bash
kubectl delete secret sh.helm.release.v1.xapp-onboarder.v1 -n ricplt
kubectl get secrets -n ricplt -l owner=helm
```
**Status**: Ongoing dependency resolution needed

---

## 7. API Testing and Health Checks

### Health Check Commands Attempted

**Monitor xApp logs**
```bash
kubectl logs -f deployment/bouncer-xapp -n ricplt
kubectl logs -f deployment/kpi-monitor-xapp -n ricplt
```

**Test Application Manager health endpoint**
```bash
kubectl exec -n ricplt deployment/deployment-ricplt-appmgr -- curl -s http://localhost:8080/ric/v1/health/ready
```

### Successfully Verified Components
- ✅ **Bouncer xApp**: Full runtime logs showing active message processing
- ✅ **Application Manager**: Pod running and healthy
- ✅ **Subscription Manager**: Pod running and healthy
- ✅ **Database Service**: StatefulSet running successfully

---

## 8. Final Deployment Summary

### Successfully Deployed
1. **Near-RT RIC Core Platform**: Partial deployment (3/6 components running)
2. **Bouncer xApp**: Fully operational with runtime logging
3. **KPI Monitor xApp**: Deployed but experiencing runtime issues
4. **Database Services**: Fully operational

### Deployment Statistics
- **Total Deployment Time**: ~2.5 hours
- **Commands Executed**: 50+ kubectl/helm/docker commands
- **Issues Resolved**: 2 major image pull issues
- **Current Running Pods**: 4/7 in healthy state
- **Helm Releases**: 3 successful deployments

### Technical Skills Demonstrated
- Container image building and management
- Kubernetes pod troubleshooting and debugging
- Helm chart deployment and lifecycle management
- Network policy and namespace configuration
- System monitoring and log analysis
- Issue resolution and systematic debugging

---



