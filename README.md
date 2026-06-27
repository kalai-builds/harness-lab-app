# Harness Lab App

A minimal Flask app used to practice Harness CI/CD pipelines against a local
Docker Desktop Kubernetes cluster.

## Endpoints
- `GET /` -> returns a JSON message, app version, and pod hostname (useful to
  confirm rolling/canary/blue-green deployments are actually swapping pods)
- `GET /health` -> health check used by K8s readiness/liveness probes

## Local test (without Harness, just to confirm the app works)
```bash
pip install -r requirements.txt
python app.py
# visit http://localhost:5000
```

## Docker build/test locally
```bash
docker build -t harness-lab-app:local .
docker run -p 5000:5000 harness-lab-app:local
```

## Kubernetes manifests
- `k8s/deployment.yaml` -- uses `<+artifact.image>` so Harness injects the
  image built by the CI stage automatically. (If deploying manually with
  kubectl instead of Harness, replace that placeholder with a real image
  tag first.)
- `k8s/service.yaml` -- exposes the app on NodePort 30080

## Manual deploy test (bypassing Harness, just to sanity check the cluster)
```bash
docker build -t harness-lab-app:v1 .
kubectl create namespace harness-lab --dry-run=client -o yaml | kubectl apply -f -
sed 's#<+artifact.image>#harness-lab-app:v1#' k8s/deployment.yaml | kubectl apply -n harness-lab -f -
kubectl apply -n harness-lab -f k8s/service.yaml
kubectl get pods -n harness-lab
curl http://localhost:30080/
```
