# E2 Simulator Integration

This document outlines the E2 simulator integration attempts during the Near-RT RIC platform deployment. The focus was on identifying, deploying, and configuring E2 simulation components to enable testing of the RIC platform's E2 interface functionality.

---

## E2 Components Status in Near-RT RIC Platform

### Deployed E2 Components via Near-RT RIC Platform

The Near-RT RIC platform deployment included several E2-related components:

#### E2 Manager (E2MGR)

**Inspect E2 Manager pod configuration**
```bash
kubectl describe pod deployment-ricplt-e2mgr-6dbf74545f-8w7pd -n ricplt
```

<img width="1594" height="763" alt="Screenshot from 2025-08-22 20-23-10" src="https://github.com/user-attachments/assets/19bcce6d-396a-48f6-b149-e00768f8ca24" />

**Component Details:**
- **Pod Name**: deployment-ricplt-e2mgr-6dbf74545f-8w7pd
- **Image**: nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-e2mgr:3.0.1
- **Ports**: 3800/TCP, 4561/TCP, 3801/TCP
- **Health Endpoints**: 
  - Liveness: `http://:3800/v1/health`
  - Readiness: `http://:3800/v1/health`

**Current Status**: ImagePullBackOff
```
Status: Pending
State: Waiting
Reason: ImagePullBackOff
```

#### E2 Termination (E2TERM)

**Inspect E2 Termination pod configuration**
```bash
kubectl describe pod deployment-ricplt-e2term-alpha-6d6bbc684d-qdchl -n ricplt
```

<img width="1181" height="785" alt="Screenshot from 2025-08-22 20-25-27" src="https://github.com/user-attachments/assets/13688a5c-e28d-4815-8747-d17d2d015158" />

**Component Details:**
- **Pod Name**: deployment-ricplt-e2term-alpha-6d6bbc684d-qdchl
- **Image**: nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-e2term-alpha
- **Status**: CrashLoopBackOff with multiple restarts

**Runtime Status:**
```
NAME                                              READY   STATUS             RESTARTS      AGE
deployment-ricplt-e2term-alpha-6d6bbc684d-qdchl   0/1     CrashLoopBackOff   7 (62s ago)   16m
```

---

## E2 Simulator Discovery and Search Efforts

### Commands Executed for E2 Simulator Search

#### Repository-wide Search for E2 Simulator Components

**Search for E2 simulator files in repository**
```bash
find ~/ric-dep -name "*e2sim*" -o -name "*simulator*" | head -10
```

**Search for E2-related directories with simulation components**
```bash
find ~/ric-dep -name "*e2*" -type d | grep -i sim
```

#### Examine repository structure for simulation components

**List repository directory contents**
```bash
ls ~/ric-dep/*/
```

<img width="1660" height="544" alt="Screenshot from 2025-08-22 22-03-00" src="https://github.com/user-attachments/assets/2b1ea5a5-7d79-47e9-b270-8345c5c334d5" />

**Search Results:**
- No dedicated E2 simulator Helm charts found
- No standalone E2 simulator deployment files discovered
- E2-related components present only as part of platform deployment

## Repository Structure Analysis

#### Examined directories for potential E2 simulator components

**Check helm directory for E2 components**
```bash
ls ~/ric-dep/helm/
```

<img width="1697" height="56" alt="Screenshot from 2025-08-22 20-41-11" src="https://github.com/user-attachments/assets/c78e1382-bfac-4c83-b9f4-5cf2ce5e3e2a" />

Output showed: e2mgr, e2term (as platform components)

**Check for simulator-specific configurations**
```bash
ls ~/ric-dep/RECIPE_EXAMPLE/ | grep -i sim
```

<img width="817" height="56" alt="Screenshot from 2025-08-22 20-42-30" src="https://github.com/user-attachments/assets/bfba7c8d-3a5e-474c-bda9-2e3ed1a176a5" />

No simulator-specific recipe files found

### Repository Content Analysis

**Available E2-Related Components:**
```
- `~/ric-dep/helm/e2mgr/` - E2 Manager Helm chart
- `~/ric-dep/helm/e2term/` - E2 Termination Helm chart
```

<img width="619" height="123" alt="Screenshot from 2025-08-22 22-12-52" src="https://github.com/user-attachments/assets/9e84fe05-1ce0-4c37-beb7-48ad258d4edf" />

- No standalone simulator components identified

---

## E2 Interface Components Configuration

### E2 Manager Configuration Analysis

#### Environment Configuration
```bash
# E2 Manager uses environment variables from:
configmap-ricplt-e2mgr-env
configmap-ricplt-dbaas-appconfig
```

#### Volume Mounts
```
/opt/E2Manager/resources/configuration.yaml (from configmap-ricplt-e2mgr-configuration-configmap)
/opt/E2Manager/router.txt (from configmap-ricplt-e2mgr-router-configmap)
```

### E2 Termination Configuration

**Container Command:** `/run_rtmgr.sh`  
**Health Check Endpoints:**
- Alive: `http://:8080/ric/v1/health/alive`
- Ready: `http://:8080/ric/v1/health/ready`

---

## Integration Issues and Challenges

### Primary Issue: Container Image Access

#### E2 Manager Image Pull Failure
```
Failed to pull image "nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-e2mgr:3.0.1": 
Error response from daemon: manifest for nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-e2mgr:3.0.1 not found: manifest unknown: manifest unknown
```

<img width="1784" height="241" alt="Screenshot from 2025-08-22 22-30-46" src="https://github.com/user-attachments/assets/2a6964fe-4c0f-40f4-8483-fead39045315" />

#### Registry Authentication Issues

**Attempt to create registry access secret**
```bash
kubectl create secret docker-registry <secret-name> \
  --docker-server=nexus3.o-ran-sc.org:10002 \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email> -n ricplt
```

<img width="869" height="114" alt="Screenshot from 2025-08-22 21-59-27" src="https://github.com/user-attachments/assets/f4de2364-c2a2-4411-ae04-91759077baf6" />

**Result**: Commands failed due to missing credentials for O-RAN SC private registry

### Secondary Issue: Missing Dedicated E2 Simulator

#### Alternative Integration Attempts

**Attempt to locate external E2 simulator**
```bash
git clone https://github.com/o-ran-sc/hello-world-xapp.git ~/hello-world-xapp
```

Multiple authentication failures:
```
Username for 'https://github.com': abhishek-8081
remote: Invalid username or token. Password authentication is not supported for Git operations.
```

<img width="1013" height="111" alt="Screenshot from 2025-08-22 22-34-26" src="https://github.com/user-attachments/assets/21a32738-bd28-433c-97c6-d919477a1a1e" />

**Result**: Unable to access O-RAN SC repositories for additional E2 simulation components

---

## Technical Analysis and Findings

### E2 Interface Architecture Discovery

Based on deployed components analysis:

1. **E2 Manager Role:**
   - Manages E2 connections and node registrations
   - Handles E2 subscription management
   - Provides REST API endpoints for E2 operations

2. **E2 Termination Role:**
   - Terminates E2 protocol connections
   - Routes E2 messages between RAN nodes and xApps
   - Implements E2 protocol stack

### Missing Components Identified

1. **Dedicated E2 Simulator**: No standalone E2 node simulator found
2. **E2 Test Harness**: No integrated testing framework discovered
3. **RAN Simulator Integration**: No obvious integration with RAN simulation tools

---

## Monitoring and Verification Commands

### E2 Component Status Monitoring

**Monitor E2 component pod status**
```bash
kubectl get pods -n ricplt | grep e2
```

**Detailed pod inspection for troubleshooting**
```bash
kubectl describe pod deployment-ricplt-e2mgr-6dbf74545f-8w7pd -n ricplt
kubectl describe pod deployment-ricplt-e2term-alpha-6d6bbc684d-qdchl -n ricplt
```

**Check E2-related services**
```bash
kubectl get svc -n ricplt | grep e2
```

### Log Analysis Attempts

**Attempt to access E2 component logs**
```bash
kubectl logs deployment-ricplt-e2mgr-6dbf74545f-8w7pd -n ricplt
kubectl logs deployment-ricplt-e2term-alpha-6d6bbc684d-qdchl -n ricplt
```

<img width="694" height="65" alt="Screenshot from 2025-08-22 22-43-00" src="https://github.com/user-attachments/assets/f7fa2daf-e0a9-4aee-89c3-04e23b1e723f" />

**Result**: Unable to retrieve logs due to ImagePullBackOff and CrashLoopBackOff states

---

## E2 Simulator Integration Status

### Current Integration State

| Component | Status | Issue | Integration Level |
|-----------|--------|-------|------------------|
| E2 Manager | ❌ Failed | ImagePullBackOff | Platform Component Only |
| E2 Termination | ❌ Failed | CrashLoopBackOff | Platform Component Only |
| Dedicated E2 Simulator | ❌ Not Found | Missing Component | Not Available |
| E2 Test Framework | ❌ Not Available | No Test Tools | Not Integrated |

### Integration Challenges Identified

1. **Registry Access**: Unable to pull O-RAN SC private registry images
2. **Missing Simulator**: No standalone E2 simulator in deployment repository
3. **Documentation Gap**: Limited documentation for E2 simulator integration
4. **Authentication Requirements**: Private registry requires O-RAN SC credentials

---

## Alternative Integration Approaches

### Potential Solutions Investigated

#### 1. External E2 Simulator Integration

**Attempted repository cloning for external simulators**
```bash
git clone https://github.com/o-ran-sc/hello-world-xapp.git
```
**Status**: Failed due to repository access issues

#### 2. Custom E2 Simulator Development
**Approach**: Create minimal E2 protocol simulator  
**Status**: Would require substantial development effort

#### 3. Mock E2 Interface Implementation
**Approach**: Implement mock E2 responses for testing  
**Status**: Feasible but limited functionality

---

## Recommendations for E2 Simulator Integration

### Immediate Actions Required

1. **Obtain O-RAN SC Registry Access:**
   - Request credentials for nexus3.o-ran-sc.org:10002
   - Configure proper image pull secrets

2. **Identify E2 Simulator Component:**
   - Research O-RAN SC E2 simulator repositories
   - Evaluate third-party E2 simulation tools

3. **Platform Stabilization:**
   - Resolve E2 Manager and E2 Termination deployment issues
   - Ensure E2 interface components are operational

### Long-term Integration Strategy

1. **E2 Test Framework Development:**
   - Implement comprehensive E2 testing capabilities
   - Create automated E2 interface validation

2. **RAN Simulator Integration:**
   - Integrate with existing RAN simulation platforms
   - Develop E2-compliant RAN node simulators

3. **CI/CD Pipeline Enhancement:**
   - Add E2 interface testing to deployment pipeline
   - Implement automated E2 simulator deployment

---

## Technical Specifications

### E2 Interface Requirements

Based on deployed component analysis:

- **Protocol**: E2 Application Protocol (E2AP)
- **Transport**: SCTP over TCP/IP
- **Message Format**: ASN.1 encoding
- **Port Configuration**: 
  - E2 Manager: 3800, 4561, 3801
  - E2 Termination: 8080, 4560, 4561

### Integration Prerequisites

1. O-RAN SC registry access credentials
2. E2 simulator component identification
3. Network connectivity configuration
4. Protocol compliance validation tools

---
## Screenshots
<img width="873" height="662" alt="Screenshot from 2025-08-23 01-58-33" src="https://github.com/user-attachments/assets/24e3aec0-5f4c-42cd-b1e3-2027abfb3287" />
<img width="480" height="103" alt="Screenshot from 2025-08-23 02-00-00" src="https://github.com/user-attachments/assets/d257e821-e21a-413d-8e50-6a0b2ceb3e48" />
<img width="641" height="353" alt="Screenshot from 2025-08-23 02-00-28" src="https://github.com/user-attachments/assets/5807a2e9-7fb9-4ea4-bb71-6ef437d7d6c7" />
<img width="640" height="405" alt="Screenshot from 2025-08-23 02-00-54" src="https://github.com/user-attachments/assets/9166c1c5-35eb-44fd-97d0-b1dac9a4c181" />
<img width="793" height="207" alt="Screenshot from 2025-08-23 02-01-24" src="https://github.com/user-attachments/assets/7f711c19-0864-412f-8b27-fc1725b9cd03" />
<img width="814" height="783" alt="Screenshot from 2025-08-23 02-01-44" src="https://github.com/user-attachments/assets/5e13af3f-532d-48a0-a067-a7fd89788511" />







## Conclusion

The E2 simulator integration attempt revealed significant challenges primarily related to container image access and missing dedicated simulation components. While the Near-RT RIC platform includes E2 Manager and E2 Termination components, they are currently non-functional due to registry access issues. A comprehensive E2 simulator integration would require addressing authentication challenges, identifying appropriate simulation tools, and potentially developing custom E2 testing frameworks.

**Current Status**: E2 simulator integration incomplete due to infrastructure and component availability limitations.

**Next Steps**: Obtain proper registry access and identify suitable E2 simulation components for full integration testing.


