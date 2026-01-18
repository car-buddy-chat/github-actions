# Self-Hosted GitHub Actions Runner Setup for carbuddy.uk

## Prerequisites

- GitHub Personal Access Token (PAT) with scopes: `repo`, `workflow`, `admin:org_hook`
- k3s cluster with kubectl access
- Docker-in-Docker service running (already deployed)

## Setup Steps

### 1. Create GitHub Personal Access Token

1. Go to: https://github.com/settings/tokens
2. Click "Generate new token" → "Generate new token (classic)"
3. Token name: `carbuddy-uk-runner`
4. Expiration: No expiration (or your preference)
5. Scopes: Select these:
   - `repo` (Full control of private repositories)
   - `workflow` (Update GitHub Action workflows)
   - `admin:org_hook` (Full control of organization hooks)
6. Click "Generate token"
7. **Copy the token** (you won't see it again!)

### 2. Create Runner Secret in k3s

```bash
kubectl create namespace github-runner

kubectl create secret generic runner-credentials \
  --from-literal=github-token=ghp_xxxxxxxxxxxxxxxxxxxx \
  --from-literal=github-repo=car-buddy-chat/carbuddy.uk \
  -n github-runner
```

Replace `ghp_xxxxxxxxxxxxxxxxxxxx` with your actual token.

### 3. Deploy Runner Pod

Create file `runner-deployment.yaml`:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: github-runner
  namespace: github-runner

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: github-runner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: github-runner
  namespace: github-runner

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: github-runner
  namespace: github-runner
  labels:
    app: github-runner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: github-runner
  template:
    metadata:
      labels:
        app: github-runner
    spec:
      serviceAccountName: github-runner
      containers:
      - name: runner
        image: myoung34/github-runner:latest
        imagePullPolicy: Always
        env:
        - name: GITHUB_OWNER
          value: "car-buddy-chat"
        - name: GITHUB_REPOSITORY
          value: "carbuddy.uk"
        - name: GITHUB_PAT
          valueFrom:
            secretKeyRef:
              name: runner-credentials
              key: github-token
        - name: RUNNER_NAME
          value: "k3s-runner-1"
        - name: RUNNER_LABELS
          value: "self-hosted,k3s"
        - name: DOCKER_HOST
          value: "tcp://dind-service.docker-daemon:2375"
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
        volumeMounts:
        - name: runner-work
          mountPath: /home/runner/work
      volumes:
      - name: runner-work
        emptyDir:
          sizeLimit: 10Gi
```

Apply the manifest:

```bash
kubectl apply -f runner-deployment.yaml
```

### 4. Verify Runner Registration

```bash
# Check if runner pod is running
kubectl get pods -n github-runner

# Check logs
kubectl logs -n github-runner deployment/github-runner -f

# Verify in GitHub UI
gh api repos/car-buddy-chat/carbuddy.uk/actions/runners
```

### 5. Test the Runner

Push a commit to trigger the workflow:

```bash
git commit --allow-empty -m "Test self-hosted runner"
git push origin main
```

Check the workflow:

```bash
gh run list -L 1
gh run view <run-id>
```

## Troubleshooting

**Runner pod stuck in CrashLoopBackOff**:
```bash
kubectl logs -n github-runner deployment/github-runner --tail=50
```

**Docker connection fails**:
```bash
# Verify DinD service is running
kubectl get svc -n docker-daemon
kubectl get pods -n docker-daemon

# Test Docker connection from runner pod
kubectl exec -n github-runner <pod-name> -- docker version
```

**Runner doesn't register**:
- Verify GitHub PAT token is correct and has proper scopes
- Check runner logs for error messages
- Verify network connectivity to github.com

## Cost Savings

By using self-hosted runners:
- **GitHub Actions Minutes**: 0 minutes consumed (self-hosted = free)
- **GitHub Actions Storage**: GitHub-hosted runners don't count
- **Estimate Savings**: $0.008 per minute × ~500 minutes/month = $4/month saved

Plus faster builds and direct cluster access!
