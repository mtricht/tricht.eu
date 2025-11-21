+++ 
draft = false
date = 2025-11-21T15:51:15+01:00
title = "Migrating from Ingress NGINX to Envoy Gateway"
slug = "migrating-from-ingress-nginx-to-envoy-gateway" 
+++

I have been using [Ingress NGINX](https://kubernetes.github.io/ingress-nginx/) since early 2021 and have always enjoyed working with it. With it being [retired](https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/) and only receiving updates until March 2026, I decided it was a good time to explore alternatives. This is my short journey of searching for a replacement.

My current ingress-nginx setup is fairly straightforward. A set of standard ingress rules and a wildcard SSL certificate managed by cert-manager. Nothing too unusual except that the wildcard certificate is a default certificate and none of the ingresses reference a certificate.

My initial goal was to keep using the Kubernetes ingress API and to only switch out the controller underneath. I evaluated the following replacements:
- [Traefik](https://traefik.io/)  
  Includes an experimental `kubernetesingressnginx` flag which honors some ingress-nginx annotations. Unfortunately I could not get it to work properly with my default SSL certificate setup.
- [Contour](https://projectcontour.io/)  
  Contour uses Envoy as a proxy under the hood. However, it still does not support a default SSL certificate (an open [github issue](https://github.com/projectcontour/contour/issues/4556) since 2022), which is a dealbreaker for my setup.
- [Cilium](https://cilium.io/)  
  I have no experience with eBPF-based networking and I sadly wasn’t able to get the ingress functionality working in my cluster.

Since none of these were a great fit, I decided to learn the "new" Kubernetes [Gateway API](https://gateway-api.sigs.k8s.io/) which is likely to replace the ingress API in the long term. One of the implementations of the Gateway API is [Envoy Gateway](https://gateway.envoyproxy.io/) which uses [Envoy](https://www.envoyproxy.io/), a high performance C++ proxy, though there are [many others](https://gateway-api.sigs.k8s.io/implementations/).

A key difference from ingress is that the Gateway API splits responsibilities into multiple manifest files, giving you more flexibility but also more YAML.
In a nutshell, my old ingress manifest file:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/allowlist-source-range: 192.168.1.0/24
    nginx.ingress.kubernetes.io/from-to-www-redirect: "true"
  name: blog
spec:
  ingressClassName: nginx
  rules:
  - host: tricht.eu
    http:
      paths:
      - backend:
          service:
            name: blog
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - www.tricht.eu
    - tricht.eu
```
is replaced with an `HTTPRoute` (other routes such as GRPCRoute, UDPRoute and TCPRoute are available):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: blog
  labels:
    private: "true"
spec:
  parentRefs:
    - name: envoy-gateway
      namespace: envoy-gateway
  hostnames:
    - "tricht.eu"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: blog
          port: 80
```
and a `Gateway`:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-gateway-class
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: envoy-gateway
  namespace: envoy-gateway
spec:
  gatewayClassName: envoy-gateway-class
  infrastructure:
    parametersRef:
      group: gateway.envoyproxy.io
      kind: EnvoyProxy
      name: envoy-proxy
  listeners:
    - name: https
      hostname: "tricht.eu"
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
          - group: ""
            kind: Secret
            name: wildcard-tricht-eu
    - name: https-subdomain
      hostname: "*.tricht.eu"
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
          - group: ""
            kind: Secret
            name: wildcard-tricht-eu
    - name: http
      hostname: "tricht.eu"
      port: 80
      protocol: HTTP
    - name: http-subdomain
      hostname: "*.tricht.eu"
      port: 80
      protocol: HTTP
```
and to redirect from www to non-www (the `nginx.ingress.kubernetes.io/from-to-www-redirect` annotation):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: www-redirect
  namespace: envoy-gateway
spec:
  parentRefs:
    - name: envoy-gateway
  hostnames:
    - "www.tricht.eu"
  rules:
    - filters:
        - type: RequestRedirect
          requestRedirect:
            scheme: https
            hostname: "tricht.eu"
            statusCode: 301
```
and to use an IP allowlist (the `nginx.ingress.kubernetes.io/allowlist-source-range` annotation):
```yaml
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: SecurityPolicy
metadata:
  name: authorization-client-ip
spec:
  targetSelectors:
  - kind: HTTPRoute
    matchLabels:
      private: "true"
  authorization:
    defaultAction: Deny
    rules:
    - action: Allow
      principal:
        clientCIDRs:
        - 192.168.1.0/24
```

As you can see the Gateway API is highly extensible. Gateway implementations such as Envoy Gateway can provide additional CRDs like `SecurityPolicy` on top of the standard objects to allow further customization.
And thankfully there is the [ingress2gateway](https://github.com/kubernetes-sigs/ingress2gateway) command-line tool to help you migrate!