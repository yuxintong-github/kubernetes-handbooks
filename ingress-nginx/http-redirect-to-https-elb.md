


## http redirect to https On EKS And ELB

The ELB is configured to use the SSL certificate. The Nginx backend isn't tls, so it's not the ordinary way to set the annotation.

### Annotations

[force-ssl-redirect](https://github.com/kubernetes/ingress-nginx/blob/nginx-0.26.1/docs/user-guide/nginx-configuration/annotations.md#server-side-https-enforcement-through-redirect) is using to enforce to HTTPS.
but it's not working, and lead to have a 308 loop problem.

loop:

CLIENT -> ELB(HTTPS) -> NGINX(HTTP) -> Redirect to HTTPS -> ELB(HTTPS) -> ...


### Solution

#### Ingress-svc

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  annotations:
    # Enable PROXY protocol
    service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
    # Specify SSL certificate to use
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:[...]
    # Use SSL on the HTTPS port
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  ports:
    - name: http
      port: 80
      # We are using a target port of 8080 here instead of 80, this is to work around
      # https://github.com/kubernetes/ingress-nginx/issues/2724
      # This goes together with the `http-snippet` in the ConfigMap.
      targetPort: 8080
    - name: https
      port: 443
      targetPort: http
```

1. We enable the PROXY protocol on the ELB by setting the service.beta.kubernetes.io/aws-load-balancer-proxy-protocol annotation.
2. The ELB is configured to use the SSL certificate on port 443 (HTTPS).
3. The non-SSL/HTTPS traffic is sent to port 8080 on Nginx instead of the default port 80. This allows us to differentiate between the traffic which was sent encrypted and the traffic which wasn't in Nginx, as the PROXY protocol doesn't allow the ELB to pass a X-Forwarded-Proto header with the requests.


#### nginx-configuration

In the nginx-configuration ConfigMap I ended up with this:

```yaml
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
data:
  use-proxy-protocol: "true"

  # Work around for HTTP->HTTPS redirect not working when using the PROXY protocol:
  # https://github.com/kubernetes/ingress-nginx/issues/2724
  # It works by getting Nginx to listen on port 8080 on top of the standard 80 and 443,
  # and making any requests sent to port 8080 be reponded do by this code, rather than
  # the normal port 80 handling.
  ssl-redirect: "false"
  http-snippet: |
    map true $pass_access_scheme {
      default "https";
    }
    map true $pass_port {
      default 443;
    }

    server {
      listen 8080 proxy_protocol;
      return 308 https://$host$request_uri;
    }
```
This does the following:

1. Enables the PROXY protocol with the use-proxy-protocol line.
2. Turns off the HTTP->HTTPS redirect globally. This is because Nginx otherwise thinks all traffic received on port 80 is made over HTTP, and will then try to redirect it to HTTPS. This is what causes the redirect loop.
3. The http-snippet contains two bits of Nginx configuration. The map statement is used to overrule the value that $pass_access_scheme otherwise get set [here](https://github.com/kubernetes/ingress-nginx/blob/da32401c665c646954f79b61e9aa60ac562eb7b7/rootfs/etc/nginx/template/nginx.tmpl#L290-L294) 

This was necessary for me as some applications behind the ingress controller needed to know if they were served over HTTP or HTTPS - either so they could enforce being served over HTTPS, or in order to be able to generate correct URLs for links and assets.
The map configured in the http-snippet is injected further down in the Nginx configuration, and tricks Nginx into thinking all connections were made over HTTPS.
The server directive sets up Nginx to listen on port 8080 as well as port 80, and any request made to that port will receive a 308 (Permanent Redirect) response, forwarding them to the HTTPS version of the URL.


#### nginx deployment

changed the ports section of the deployment form this:

```yaml
ports:
  - name: http
    containerPort: 80
  - name: https
    containerPort: 443
```

to this:

```yaml
ports:
  - name: http
    containerPort: 80
  - name: http-workaround
    containerPort: 8080
```


