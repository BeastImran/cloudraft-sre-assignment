# 🔍 SRE Challenge: Memory Leak Debugging & Resolution

## 🐛 Issue Identification

During deployment of the application in a local Kind cluster with ArgoCD, the application exhibited abnormal memory consumption patterns. Investigation revealed a hidden intentional memory leak triggered by requests to the `/counter` endpoint.

### 🕵️ Root Cause Analysis

- Application contains a hidden memory leak mechanism inside `resources.dat`
- The file is base64-encoded Python code
- Code is dynamically decoded and executed when `/counter` endpoint receives HTTP requests
- The executed code spawns periodic processes that consume memory without proper cleanup holding/consuming all of the available memory in it's global_memory list leading to memory resource exhaustion in whole cluster.

![resource.dat file decoded. The intentional memory leak base64 encoded python code](/assets/images/resource.dat%20decoded.png "resource.dat decoded")

## 🛠️ Technical Solution

### Immediate Mitigation

1. **Resource Limiting**
   - Apply Kubernetes resource limits via deployment manifest:
     ```yaml
     resources:
       limits:
         memory: "512Mi"
         cpu: "500m"
       requests:
         memory: "256Mi"
         cpu: "250m"
     ```
   - Configure `OOMKillDisable: false` to ensure container restart on memory exhaustion

   ![OOM Killed](/assets/images/App%20killed%20OOM.png "Application OOM killed because of resource limits enforcement")

2. **Monitoring Setup**
   - Deploy Prometheus & Grafana for real-time memory monitoring
   - Create alerts for memory threshold violations (>80% usage)

### Root Cause Elimination

1. **Code Analysis**
   - Decode the `resources.dat` file:
     ```bash
     cat resources.dat | base64 -d > decoded_resources_dat.py
     ```
   - Patch the application to remove the malicious code execution path

2. **Secure Rebuild**
   - Create a secure Dockerfile with proper security scanning
   - Implement file integrity checks to detect unauthorized modifications

## 🔒 Security Practices & Prevention

1. **Container Security**
   - Use distroless/minimal base images
   - Implement read-only filesystem where possible
   - Run containers with non-root users
   - Apply Pod Security Policies (PSP) or Pod Security Standards (PSS)

2. **CI/CD Security**
   - Implement container image scanning (Trivy, Clair)
   - Add SAST (Static Application Security Testing) to pipeline
   - Set up regular security audits for base64-encoded or obfuscated content

3. **Runtime Protection**
   - Deploy Falco (or similar tools) for runtime security monitoring
   - Implement network policies to restrict unnecessary communication
   - Use admission controllers to enforce security policies

4. **Observability Enhancement**
   - Set up detailed application metrics (JVM for Java apps, custom metrics for others)
   - Implement distributed tracing for better request visibility
   - Configure log aggregation with pattern detection for suspicious activities

## 🔄 Long-term Recommendations

1. **Implement Chaos Engineering**
   - Regular chaos tests to validate resilience against resource exhaustion
   - Simulate memory leaks in controlled environments for response practice

2. **Automated Response**
   - Create runbooks for automated remediation
   - Implement Kubernetes Horizontal Pod Autoscaler (HPA) based on custom metrics

3. **Code Review Process**
   - Enhance code review to identify suspicious patterns (dynamic code loading, base64 encoding)
   - Implement "four eyes" principle for security-critical changes
