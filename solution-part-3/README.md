
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
To deploy after cloning this repo
```
kubectl apply -f kubernetes_hackathon/solution-part-3/products-db -f kubernetes_hackathon/solution-part-3/products-api  -f kubernetes_hackathon/solution-part-3/stock-api -f kubernetes_hackathon/solution-part-3/web
```
