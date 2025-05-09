

Everything works in the same way:

> Browse to http://widgetario.local to see the app

> See the API response at http://api.widgetario.local/products

Cleanup:

```
kubectl delete all,statefulset,pvc,secret,configmap -l kubernetes.courselabs.co=hackathon
```
