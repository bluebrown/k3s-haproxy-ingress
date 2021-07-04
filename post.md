Routing external traffic to the various pods in a cluster is one of the major problems when it comes to container orchestration.

One strategy is to map service ports against static ports on the host, but as the number of deployed container grows, this becomes more and more unmanageable.

If, on the other hands, dynamic ports are used, it becomes difficult to know which port corresponds to which backend.

Another problem with both static and dynamic IPs is that, when using something like an external loadbalancer, one load balancer per service would be required, which can be quite expensive.

Additionally, default service load balancing happens on layer 4. So it is not possible to inspect and manipulate the HTTP headers.

## Ingress Controller

The ingress controller was created to address the former mentioned needs. It is essentially just a reverse proxy which resolves the various DNS names within the cluster based on ingress rules. Additionally, they can operate on layer 7.

That creates a single entry point into the cluster with more powerful load balancing capabilities than the one of services. For exampling matching the host name and request path, whitelisting IPS, providing session persistence & affinity, TLS, etc.

With an ingress controller, only a single external loadbalancer is required to target the ingress controller. That lowers the operational cost.

Ingress controller are deployed like any other deployment. Its just some container with the specialized controller software inside. For example, nginx, HAProxy or traffic.

Below is the HAProxy ingress controller. HAProxy itself is a powerful software which can operate in HTTP and TCP mode. <https://github.com/haproxytech/kubernetes-ingress/tree/master/deploy>

```yaml
containers:
- name: haproxy-ingress
  image: haproxytech/kubernetes-ingress
```

## Ingress Rules

Once an ingress controller has been deployed. Familiar looking `Ingress rules` are created to hook into the request and routing flow.

These rule can match things like the `request path` and 'host name', and route the traffic to a number of backends `backend` such as the service `service`.

<https://www.haproxy.com/documentation/kubernetes/latest/configuration/ingress/>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  labels:
    name: web-ingress

  # proxy rules as known from haproxy wrapped in annotations
  annotations:
    haproxy.org/allowlist: "192.168.1.0/24, 192.168.2.100"
    haproxy.org/ssl-redirect: "true"
    haproxy.org/ssl-certificate: "default/my-tls-object"
    haproxy.org/ssl-redirect-code: "301"
    haproxy.org/rate-limit-requests: 10
    haproxy.org/rate-limit-period: "1m"
    haproxy.org/load-balance: "leastconn"
    haproxy.org/cookie-persistence: "JSESSIONID"
    haproxy.org/cors-allow-methods: "GET, POST"
    haproxy.org/path-rewrite: /foo/(.*) /\1
    haproxy.org/request-set-header: |
      Ingress-ID abcd123
      Another-Header 12345
    haproxy.org/response-set-header: |
      Cache-Control "no-store,no-cache,private"
      Strict-Transport-Security "max-age=31536000"

# define what backend should be matched
spec:
  rules:
  - host: <your-domain>
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: app
            port: 
              number: 80
```

For the HAProxy ingress controller, many options can also be set as default and or per namespace via `configmap`. That allows to encapsulate and decouple common rules. <https://www.haproxy.com/documentation/kubernetes/latest/configuration/configmap/>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubernetes-ingress
  namespace: default
# create rules for the default namespace
data:
  allowlist: "192.168.1.0/24, 192.168.2.100"
  global-config-snippet: |
    ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11
    ssl-default-bind-ciphers TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256
    tune.ssl.default-dh-param 2048
    tune.bufsize 32768
  ssl-redirect: "true"
  ssl-certificate: "default/my-tls-object"
  ssl-redirect-code: "301"
  timeout-connect: 5s
  timeout-http-request: 5s
  timeout-http-keep-alive: 5s
```

## Motivation

This pattern is not unique to Kubernetes. For example, if you have ever set up service discovery and load balancing for docker or docker swarm, you may be familiar with the objective and even some implementation details.
Most if not all the ingress controller are actually normal software loadbalancer in a Kubernetes friendly padding.

Essentially Kubernetes understood that this a common need and made it first class citizen called *ingress controller*. So instead of everyone hand rolling their own reverse proxy for the cluster, there is a common pattern with native support such as declarative object tracking.

You can take a look at this post, which goes through many aspects of an ingress controller while never actually using the term ingress controller.

{% link <https://dev.to/codingsafari/docker-service-discovery-loadbalancing-strategies-43pk> %}
