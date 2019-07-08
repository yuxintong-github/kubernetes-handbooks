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

### 2. ELB of AWS

As the way before I use nginx ingress to Expose host port(80/443),But in Aws we use ELB to expose the NGINX Ingress controller behind a Service of Type=LoadBalancer. 


```console
https://github.com/kubernetes/ingress-nginx/blob/bc4902ec70/docs/deploy/index.md#aws
```


#### Modification

##### 1. certification

As the doc said, i replacing the certification as my domain while using HTTPS

##### 2. service.beta.kubernetes.io/aws-load-balancer-backend-protocol

because of we use websocket as one backend, so rewrite *http* To *tcp*

### 3. AWS ROUTE 53

add a DNS CNAME to **ingress-nginx** svc's **EXTERNAL-IP**


### 4. Ingress's Example

This is the yaml which use the **auth_request** in nginx with the annotations

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: xxxx
  namespace: xxx
  annotations:
    nginx.ingress.kubernetes.io/auth-url: http://xxx.xxxx.svc.cluster.local:xxxx/api/v1/auth
    nginx.ingress.kubernetes.io/auth-method: POST
spec:
  rules:
  - host: xxx.xxx
    http:
      paths:
      - backend:
          serviceName: xxxx
          servicePort: 10000
        path: /
```
