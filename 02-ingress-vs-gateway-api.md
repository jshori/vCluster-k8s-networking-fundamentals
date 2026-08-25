# From Ingress to Gateway API: The Full Story

*This guide assumes you're familiar with the basics of what an API is, what a request/response cycle looks like, and how a client talks to a server. If any of that isn't clear yet, start with the "What is an API" guide first.*

## 1. Basics: What are a Resource, a Resource Type, and a CRD

*In the "What is an API" guide, we understood that Kubernetes has an API server, and that resources are the "things" you create and manage through it, like `Pod` and `Service`.*

In Kubernetes, anything that Kubernetes tracks or manages is called a **"resource"**, think of it like a record in a database. Every resource is created by following a **"resource type"** (a blueprint or category, such as `Pod` or `Service`).

`Ingress` was also a built-in Kubernetes resource type. It described how traffic coming from outside the cluster should be routed to a service inside it, using basic rules like "send requests for this domain name to this service." Kubernetes provided this natively, and it was common to everyone.

A **CRD (Custom Resource Definition)** is a mechanism for creating a new, custom resource type, one that goes beyond Kubernetes' built-in resource types (`Pod`, `Service`, `Ingress`, etc.). Any vendor or company can use a CRD to teach Kubernetes about new "resource types" that Kubernetes didn't know about out of the box.

## 2. The old way: every vendor built its own CRDs

Before the Gateway API standard existed, Kubernetes only had one basic resource type for this purpose, called `Ingress`. But `Ingress` was quite limited, so every vendor, Traefik, NGINX, and others, built their own extra resource types via CRDs, to offer advanced features that plain `Ingress` couldn't provide.

When a user wanted to use Traefik, and installed it on their Kubernetes cluster, the CRDs that the Traefik vendor had built (`IngressRoute`, `IngressRouteTCP`, `Middleware`, etc.) would get installed automatically, bundled right in with the Helm chart.

This was the **old approach**: one basic, common `Ingress` resource type for everyone, but for advanced functionality, every vendor had its own separate set of extra resource types (CRDs), understood only by that vendor's own software.

## 3. The struggle set in, and something new was needed

Early on, people were happy with the `Ingress` API. It was new, and it solved the basic use case (routing external traffic to a service). But gradually, as people started using Kubernetes for larger, more complex applications, they began to feel the strain.

The limitations of `Ingress` became apparent. It only supported HTTP/HTTPS. It had no built-in way to do things like traffic splitting (sending, say, 90% of traffic to one version of an app and 10% to a newer version, for testing) or routing based on request headers (like sending traffic to a different service depending on which mobile app version made the request). Every vendor had its own annotations/CRDs to work around this, which meant switching from one controller to another was difficult.

On top of that, `Ingress` mixed everything into one resource. TLS settings, load balancer configuration, and routing rules for an application all lived in the same object. There was no separate resource for "the entry point" versus "an application's routes."

For example, a single `Ingress` object typically looked like this:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  tls:
  - hosts:
    - foo.example.com
    secretName: example-com   # TLS certificate, an infrastructure concern
  rules:
  - host: foo.example.com
    http:
      paths:
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: foo-orders-app   # routing to a Service, an application concern
            port:
              number: 80
```

Notice the `tls` block (which certificate to use, an infrastructure-level decision) and the `rules` block (which path goes to which app, an application-level decision) both live in this exact same object. There's no way to give one team edit access to just the `tls` part and another team edit access to just the `rules` part, it's all one resource. This made it hard to cleanly split responsibilities between the team managing shared infrastructure and the team managing individual applications.

Kubernetes itself is a graduated CNCF (Cloud Native Computing Foundation) project, where developers from around the world contribute for free to build and improve it. A specific team within Kubernetes, called **SIG-Network**, recognized that the limitations of `Ingress` couldn't be fixed with small patches, something new was needed. So they began designing a new standard, and this time, they brought vendors like Traefik and Istio into the discussion from the very start, so the new standard would be shaped around everyone's real-world use cases.

This is how `Gateway API` came to be, the official successor to `Ingress`.

## 4. Timeline: what happened, and when

- **2019**: SIG-Network began work on this new standard (discussions started ahead of KubeCon San Diego 2019)
- It was initially named **"Service APIs"**; in **February 2021** it was renamed to **"Gateway API"**
- For several years it remained in an "Experimental/Alpha" stage, usable, but without a stability guarantee. During this time, companies like Istio, Envoy Gateway, Cilium, and GKE gradually added support for it
- **October 31, 2023**: Gateway API reached its **v1.0 GA (General Availability)** release, meaning it was now officially declared "production-ready and stable." At the same time, the older `Ingress` API was officially "frozen", no new features are added to it anymore, it stays as-is. All new development now happens in Gateway API
- In **2022**, a separate workstream started, called **GAMMA** (Gateway API for Mesh Management and Administration), whose job was to figure out whether Gateway API could also be used for service mesh (east-west traffic)
- **May 9, 2024**: Gateway API **v1.1** was released, bringing service mesh support (the result of GAMMA's work) and GRPCRoute officially into the Standard/GA channel
- **February 27, 2026**: Gateway API **v1.5** was released, moving `TLSRoute` (among other features) into the Standard/GA channel
- **June 30, 2026**: Gateway API **v1.6** was released, moving `TCPRoute` and `UDPRoute` into the Standard/GA channel as well, so raw TCP and UDP traffic routing now has the same production-grade stability as HTTP and TLS routing. As of this guide, **v1.6.1** (a bug-fix patch release) is the latest available version. By the time of the original GA release (November 2023), Gateway API already had 20+ different companies/projects implementing it, Envoy Gateway, Istio, Cilium, GKE, Kong, and many more; that number has grown further since
- **March 2026**: Per the Kubernetes SIG Network announcement, the `ingress-nginx` controller has **already stopped** receiving new releases, bug fixes, or security patches after this date (best-effort maintenance only ran until then). This makes migration away from it not just a "nice to have" but a necessary step for teams still relying on it

**Short summary:** Limitations of Ingress became apparent, so SIG-Network designed a new standard together with vendors (about 4 years of work, 2019 to 2023). It reached GA in 2023, and Ingress was frozen. Mesh support was added too (GAMMA, 2022 to 2024). By mid-2026, every major traffic type, HTTP, TLS, TCP, and UDP, has reached full production-grade (Standard/GA) status. Today it's the industry standard, and the retirement of older systems like `ingress-nginx` has made adopting it even more necessary.

## 5. The new approach: how these CRDs are different

When people install a vendor based on the Kubernetes Gateway API standard (like Traefik), the new CRDs that get installed, `Gateway`, `GatewayClass`, `HTTPRoute`, `TLSRoute`, are fundamentally different from the older CRDs (like `IngressRoute`, `Middleware`, which Traefik built for itself).

The difference: the older CRDs (`IngressRoute`, etc.) were built solely by Traefik, solely for Traefik's own use, no other vendor (NGINX, Envoy Gateway) understood them at all. The new CRDs (`Gateway`, `HTTPRoute`, `TLSRoute`), on the other hand, were built by Kubernetes' own SIG-Network group, as a shared, common standard.

Being a "shared standard" doesn't just mean different vendors happen to build similar things. In practice, every vendor installs the exact same, official YAML file published by SIG-Network, without modifying it:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.6.1/standard-install.yaml
```

In other words, the CRD's "shape/schema" (the blueprint of the form) is literally identical for everyone, coming from a single source. The **controller**, the software that reads this form and does the actual work, is what each vendor writes independently. If the vendor is Traefik, that's Traefik's own controller. If the vendor is Envoy Gateway, that's the Envoy Gateway controller. The schema is shared, the implementation is not.

This is why, when Traefik installs its Gateway API support, it isn't creating its own new CRDs. It's using the same official CRDs that are the Kubernetes standard, ones that every Gateway API-compatible vendor in the world (Traefik, Envoy Gateway, Istio) already understands in exactly this same form.

This is also why, if you switch from Traefik to Envoy Gateway (or any other Gateway API vendor) tomorrow, your `Gateway`/`TLSRoute` YAML mostly stays the same, only the `controllerName` in `GatewayClass` changes (which tells Kubernetes which vendor will actually implement this Gateway).

## Sources

- [Traefik Helm Chart, CRD management](https://github.com/traefik/traefik-helm-chart)
- [Kubernetes Gateway API, official docs](https://gateway-api.sigs.k8s.io/)
- [Migrating from Ingress, Gateway API official docs](https://gateway-api.sigs.k8s.io/guides/getting-started/migrating-from-ingress/)
- [The Story of Gateway API, Google Open Source Blog](https://opensource.googleblog.com/2023/11/the-story-of-gateway-api.html)
- [Gateway API v1.0: GA Release, Kubernetes blog](https://kubernetes.io/blog/2023/10/31/gateway-api-ga/)
- [Gateway API v1.1: Service mesh, GRPCRoute, Kubernetes blog](https://kubernetes.io/blog/2024/05/09/gateway-api-v1-1/)
- [Gateway API v1.5: Moving features to Stable, Kubernetes blog](https://kubernetes.io/blog/2026/04/21/gateway-api-v1-5/)
- [Gateway API v1.6: TCPRoute and UDPRoute Graduate to Standard, Kubernetes blog](https://kubernetes.io/blog/2026/08/03/gateway-api-v1-6-release/)
- [GAMMA Initiative, official docs](https://gateway-api.sigs.k8s.io/mesh/gamma/)
- [kubernetes-sigs/gateway-api, GitHub (rename history)](https://github.com/kubernetes-sigs/gateway-api)
- [Ingress NGINX retirement, Kubernetes SIG Network announcement](https://techcommunity.microsoft.com/blog/azurearchitectureblog/from-ingress-to-gateway-api-a-pragmatic-path-forward-and-why-it-matters-now/4489779)
