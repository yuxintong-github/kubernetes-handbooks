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

##### 1. single-certificate

As the doc said, i replacing the certification as my domain while using HTTPS

`
service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-west-2:xxxxx:certificate/xxxxxxx
`

##### 2. Multi-certificates

should create a new ingress-svc to support multi-certs.

```
kind: Service
apiVersion: v1
metadata:
  name: ingress-nginx-new-domain-name
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  annotations:
    # replace with the correct value of the generated certificate in the AWS console
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:us-west-2:XXXXXXXX:certificate/XXXXXX-XXXXXXX-XXXXXXX-XXXXXXXX"
    # the backend instances are HTTP
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
    # Map port 443
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "https"
    # Ensure the ELB idle timeout is less than nginx keep-alive timeout. By default,
    # NGINX keep-alive is set to 75s. If using WebSockets, the value will need to be
    # increased to '3600' to avoid any potential issues.
    service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-internal: "0.0.0.0/0"
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  ports:
    - name: https
      port: 443
      targetPort: http
```


##### 3. service.beta.kubernetes.io/aws-load-balancer-backend-protocol

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
