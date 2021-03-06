[AWS Service annotations](https://github.com/kubernetes/kubernetes/blob/v1.12.0/pkg/cloudprovider/providers/aws/aws.go)
---

- `service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval` (in minutes)
- `service.beta.kubernetes.io/aws-load-balancer-access-log-enabled` (true|false)
- `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name`
- `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix`
- `service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags` (comma-separated list of key=value)
- `service.beta.kubernetes.io/aws-load-balancer-backend-protocol` (http|https|ssl|tcp)
- `service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled` (true|false)
- `service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout` (in seconds)
- `service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout` (in seconds, default 60)
- `service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled` (true|false)
- `service.beta.kubernetes.io/aws-load-balancer-extra-security-groups` (comma-separated list)
- `service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold`
- `service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval`
- `service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout`
- `service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold`
- `service.beta.kubernetes.io/aws-load-balancer-internal` (true|false)
- `service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: '*'`
- `service.beta.kubernetes.io/aws-load-balancer-ssl-cert` (IAM or ACM ARN)
- `service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy`
- `service.beta.kubernetes.io/aws-load-balancer-ssl-ports` (default '*')
- `service.beta.kubernetes.io/aws-load-balancer-type: nlb`

// ServiceAnnotationLoadBalancerExtraSecurityGroups is the annotation used
// on the service to specify additional security groups to be added to ELB created
- const ServiceAnnotationLoadBalancerExtraSecurityGroups = "service.beta.kubernetes.io/aws-load-balancer-extra-security-groups"

// ServiceAnnotationLoadBalancerSecurityGroups is the annotation used
// on the service to specify the security groups to be added to ELB created. Differently from the annotation
// "service.beta.kubernetes.io/aws-load-balancer-extra-security-groups", this replaces all other security groups previously assigned to the ELB.
- const ServiceAnnotationLoadBalancerSecurityGroups = "service.beta.kubernetes.io/aws-load-balancer-security-groups"
