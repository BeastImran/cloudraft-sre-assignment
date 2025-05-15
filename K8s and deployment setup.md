# Metrics-App Deployment in Kind Cluster with ArgoCD

## 📋 Implementation Overview

The `metrics-app` deployment leverages GitOps principles using ArgoCD for continuous delivery in a local Kind cluster. The application is containerized with security-hardened configurations and follows Kubernetes best practices.


## 🔧 Architecture Components

- **Helm Chart**: Custom chart with proper templating for `metrics-app`
- **ArgoCD**: Declarative deployment via Application CRD with sync policies
- **Ingress**: NGINX ingress controller for external endpoint exposure
- **Network Policies**: Granular traffic control for pod communication
- **HPA**: Auto-scaling based on CPU/Memory metrics

## 🛡️ Security Implementation

| Feature | Implementation |
|---------|----------------|
| Pod Security | Non-root user, read-only filesystem, runtime seccomp profile |
| Network Security | Ingress/Egress policies with explicit allowed sources |
| Secret Management | `MYPASSWORD` passed via ArgoCD parameter substitution |
| Linux Capabilities | Dropped ALL capabilities for least privilege |
| Security Headers | CSP, X-XSS-Protection via Ingress annotations |

## 🚀 Deployment Strategy

The deployment uses a controlled rolling update strategy with:
- `maxSurge: 1` & `maxUnavailable: 0` for zero-downtime updates

![Gradual rollout deployment strategy](/assets/images/App%20deployment%20Gradual%20Rollout.png)

- `minReadySeconds: 30` to ensure stability before proceeding
- Pod anti-affinity for high availability across nodes
- Graduated sync via ArgoCD sync-waves (-2 → 3)

**Git Security**:
   - Implemented pre-commit hooks for code quality and security:
     ```yaml
     # Key hooks configured:
     - trailing-whitespace, end-of-file-fixer       # Code cleanliness
     - detect-aws-credentials, detect-private-key   # Credential scanning
     - gitleaks                                     # Secret detection
     - talisman-commit, talisman-push, trufflehog   # Security scanning
     - biome check                                  # Code formatting
     ```
   - Layered security approach prevents secrets and credentials from entering repository
   - Automatic checks for large files, symlinks, and executable permissions

        ![Pre-commit git hooks in action](/assets/images/pre-commit%20git%20hooks%20running.png)

## 🚦 Monitoring & Observability

### Current Gaps

- Missing alerts for memory/CPU threshold violations
- No distributed tracing for latency analysis
- Absence of centralized logging

1. **Metrics Stack**:
   - Deploy Prometheus + Grafana for resource monitoring
   - Create SLO/SLI dashboards with key metrics

    ![Prometheus + Grafana monitoring](/assets/images/App%20Grafana%20monitoring.png)

### Proposed Enhancements

2. **Logging**:
   - Implement EFK/PLG stack for centralized logging
   - Configure log retention policies and structured logging

3. **Alerts**:
   - Memory usage threshold alerts
   - Error rate and latency SLO violations
   - Pod restart notifications

## 🔒 Additional Security Improvements

1. **Container Hardening**:
   - Implement distroless base images
   - Add runtime vulnerability scanning (Falco)
   - Configure admission controllers (OPA/Gatekeeper)

2. **RBAC Enhancement**:
   - Implement fine-grained RBAC for service account
   - Use Kubernetes secrets externally managed (Vault integration)

3. **Network Security**:
   - Implement mTLS with service mesh
   - Add egress filtering for all outbound traffic

## 📈 Future Scalability Considerations

- Implement vertical pod autoscaler for resource optimization
- Add custom metrics for autoscaling (request latency)
- Implement multi-region deployment for disaster recovery
- Configure pod topology spread constraints for zone-aware scheduling
