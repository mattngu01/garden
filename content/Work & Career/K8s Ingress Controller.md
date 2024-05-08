This is a separate pod / application (load balancer? reverse proxy?) that routes traffic from the cluster to the respective service, and subsequently a pod. This is not a part of a K8s cluster by default.

When you define an `Ingress` resource, this will adjust the ingress controller's `ConfigMap` containing rules & configurations to route requests accordingly to a [[K8s Service]].

Here is an example flow:

```
client -> ingress controller -> service -> pod(s)
```

## Resources
- https://kubernetes.io/docs/concepts/services-networking/ingress/
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/