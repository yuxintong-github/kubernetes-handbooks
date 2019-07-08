## Ingress-nginx && Aws-eks handbooks
deploy ingress-nginx on aws-eks environment instead of service-loadbalancer

### 1. Deploy ingress-nginx

Official address

```console
https://github.com/kubernetes/ingress-nginx/blob/bc4902ec70/docs/deploy/index.md#prerequisite-generic-deployment-command
```

#### Modification

-  Deployment->DaemonSet
-  Add nodeselector

```console
./mandatory_ds.yaml
```



