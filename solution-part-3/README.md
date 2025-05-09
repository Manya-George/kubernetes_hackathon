
Delete the previous database - you can't change a Deployment into a StatefulSet, or change a Service to have no ClusterIP:

```
kubectl delete deploy products-db

kubectl delete svc products-db

# you may have some PVCs lingering from the labs:
kubectl delete pvc -l app=products-db
```

Restart Pods with changed config:

```
kubectl rollout restart deploy/products-api deploy/stock-api
```
