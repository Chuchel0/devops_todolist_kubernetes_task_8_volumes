# Validation instructions

Deploy everything first:

```bash
bash bootstrap.sh
```

Pick one pod for the checks below:

```bash
POD=$(kubectl get pods -n todoapp -l app=todoapp -o jsonpath='{.items[0].metadata.name}')
```

## 1. App is running

```bash
kubectl get pods -n todoapp            # STATUS Running, READY 1/1
kubectl get pv pv-data                 # STATUS Bound
kubectl get pvc -n todoapp             # pvc-data Bound
kubectl get svc -n todoapp
```

Open the app (Minikube): `minikube service todoapp-nodeport -n todoapp`
or `http://<node-ip>:30007`. The health endpoint should answer 200:

```bash
kubectl exec -n todoapp $POD -- curl -s -o /dev/null -w "%{http_code}\n" localhost:8080/api/health
```

## 2. ConfigMap data is mounted as files

```bash
kubectl exec -n todoapp $POD -- ls -l /app/configs
kubectl exec -n todoapp $POD -- cat /app/configs/PYTHONUNBUFFERED   # prints: 1
```

Read-only check (must fail with "Read-only file system"):

```bash
kubectl exec -n todoapp $POD -- touch /app/configs/test
```

## 3. Secret data is mounted as a file

```bash
kubectl exec -n todoapp $POD -- ls -l /app/secrets
kubectl exec -n todoapp $POD -- cat /app/secrets/SECRET_KEY
kubectl exec -n todoapp $POD -- touch /app/secrets/test   # must fail (read-only)
```

## 4. Persistent volume (optional)

```bash
kubectl exec -n todoapp $POD -- sh -c 'echo ok > /app/data/test.txt'
kubectl delete pod -n todoapp $POD
# after the new pod is Running:
NEW=$(kubectl get pods -n todoapp -l app=todoapp -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n todoapp $NEW -- cat /app/data/test.txt     # prints: ok
```
