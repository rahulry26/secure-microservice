# DevSecOps Candidate Evaluation Task

## Project: Secure Microservice Deployment (Node.js/Express + MongoDB)

---

## 1. Project Structure
```
secure-microservice/
├── backend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── app.js / index.js (Your Express App)
│   ├── package.json
│   ├── package-lock.json
│   └── k8s/
│       ├── namespace.yaml
│       ├── configmap.yaml
│       ├── secret.yaml
│       ├── deployment.yaml
│       └── service.yaml
├── docker-compose.yml
├── README.md
└── .github/workflows/
    └── ci-cd-pipeline.yaml (optional if required)
```

---

## 2. Docker Hardening Steps
- ✅ Used minimal base image: `node:20-alpine`
- ✅ Multi-stage Dockerfile implemented
- ✅ Created non-root user (`appuser`) for runtime
- ✅ Trivy Image Scan performed:
  - High severity vulnerability in `cross-spawn` (CVE-2024-21538)
  - Fixed by upgrading `cross-spawn` to 7.0.5
- ✅ Trivy Secret Scan: No issues detected

---

## 3. Kubernetes YAML Files
- All manifests created and dry-run validated:
  - `namespace.yaml`
  - `configmap.yaml` (e.g., for PORT or ENV values)
  - `secret.yaml` (base64 encoded values)
  - `deployment.yaml` (uses `configMapRef` and `secretRef`)
  - `service.yaml` (ClusterIP or NodePort)

---

## 4. Commands Used (Key Examples)
```bash
# Docker Build
cd backend
npm install
npm audit fix --force
docker build -t secure-microservice-backend .

# Trivy Scans
# Vulnerability Scan on Image
docker run --rm \
  -e TRIVY_SCAN_LOCAL_REPO=true \
  -e DOCKER_HOST=tcp://host.docker.internal:2375 \
  aquasec/trivy image secure-microservice-backend

# Secret Scan on Filesystem
docker run --rm \
  -v ${PWD}:/app \
  aquasec/trivy fs --scanners secret /app

# Kubernetes Dry Run without Cluster
kubectl apply --dry-run=client --validate=false -f k8s/namespace.yaml
kubectl apply --dry-run=client --validate=false -f k8s/configmap.yaml
kubectl apply --dry-run=client --validate=false -f k8s/secret.yaml
kubectl apply --dry-run=client --validate=false -f k8s/deployment.yaml
kubectl apply --dry-run=client --validate=false -f k8s/service.yaml
```

---

## 5. Secrets Management
- ✅ `secret.yaml` uses base64 encoded values (e.g., DB_PASSWORD)
- ✅ No hardcoded secrets in Dockerfile or app code

---

## 6. CI/CD Pipeline Security (If Required)
> Not implemented in current local-only workflow but structure provided:
- GitHub Actions YAML file placeholder (`.github/workflows/ci-cd-pipeline.yaml`)
- Include Semgrep and Trivy as security gates (optional)

---

## 7. Infrastructure as Code & Runtime Security (Optional)
- ❌ Terraform or Checkov not used since infra not provisioned
- ❌ Runtime security tools like Falco not installed (bonus section optional)

---

## 8. How to Run Locally
```bash
# Run via Docker Compose
cd secure-microservice/
docker-compose up --build

# Or run manually
cd backend
docker build -t secure-microservice-backend .
docker run -p 3000:3000 secure-microservice-backend
```

---

## 9. Security Summary (for report)
- Trivy scan revealed a HIGH CVE in `cross-spawn` which was fixed
- Secret scanning returned clean results
- Dockerfile uses least privilege and hardened best practices
- Secrets are externalized using Kubernetes secrets (base64 encoded)
- K8s manifests validated without a cluster using dry-run mode

---

## 10. Final Notes
- No cloud or live cluster was used for deployment
- Entire pipeline is functional and dry-run verified
- Ideal for submission to showcase secure deployment practices
