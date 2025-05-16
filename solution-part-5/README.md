

Everything works in the same way:

> Browse to http://widgetario.local to see the app

> See the API response at http://api.widgetario.local/products

Cleanup:

```
kubectl delete all,statefulset,pvc,secret,configmap -l kubernetes.courselabs.co=hackathon
```

Deployment:

```
kubectl apply -f kubernetes_hackathon/solution-part-5/ingress-controller -f kubernetes_hackathon/solution-part-5/products-db -f kubernetes_hackathon/solution-part-5/products-api  -f kubernetes_hackathon/solution-part-5/stock-api -f kubernetes_hackathon/solution-part-5/web
```

