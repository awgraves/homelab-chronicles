---
weight: 1
title: "Ch 8: Deploy & Expose"
bookToc: true
type: "docs"
bookFlatSection: true
---

# Deploy & Expose

Ready to deploy & expose my first app on the cluster. 🚀

This [homepage app](https://gethomepage.dev/) seems like a good candidate - simple yet useful.

## Installation

The [documentation](https://gethomepage.dev/installation/k8s/#service) includes example manifest files I can apply to my cluster.

I'll create and commit them in my git repo at `/kubernetes/homepage`.

### Service Account

Kubernetes has both 'user' and 'service' accounts. 

These provide identities when accessing API resources.

I create a [service account](https://kubernetes.io/docs/concepts/security/service-accounts/) for the homepage service.

![service account yaml](/deploy-expose/service-account.png)

### Secret

This particular [secret](https://kubernetes.io/docs/concepts/configuration/secret/) is a service account API access token.

The value of the secret will be generated and managed by the cluster automatically.

![secret token](/deploy-expose/token-secret.png)

### ClusterRole & ClusterBinding

Together, these provide [role based access control (RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) for the service account.

Here, these values allow read access to certain resources on the cluster.

![cluster role](/deploy-expose/cluster-role.png)

![cluster role binding](/deploy-expose/cluster-role-binding.png)

### ConfigMap

A [configmap](https://kubernetes.io/docs/concepts/configuration/configmap/) is a way to provide non-secret values to pod(s) in the cluster.

In this case, its mostly specifying which links should appear on the homepage.

I'll leave in the placeholders for now:

![configmap](/deploy-expose/config-map.png)

### Deployment

A [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) defines a pod template and how many replicas to create.

The pod template is also were the container(s) to be run on the pod are specified.

I update the allowed hosts env var on the homepage container to be `k3s.homelab.lan`.

![deployment](/deploy-expose/deployment.png)

### Service

A [service](https://kubernetes.io/docs/concepts/services-networking/service/) is a way to group pods together and provide a single entrypoint to them.

This abstracts away the underlying pods since they can die and be replaced with different IPs at any given time.

The number of pod replicas might also scale up or down, and the service abstracts that away as well.

![service](/deploy-expose/service.png)

The type [ClusterIP](https://kubernetes.io/docs/concepts/services-networking/service/#type-clusterip) means this service is only accessible within the cluster.

There is second type called [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) which would bind the service onto a high numbered port on the hosts (nodes).

This could work, but comes at the cost of complexity needing to recall which port a given service is accessed over.

The third type, [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer) is supported in cloud environments, but I'm not on the cloud.

Luckily, there's another option...

### Ingress Controller

An [IngressController](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) behaves similarly to a reverse proxy running inside the cluster.

It can listen to incoming traffic to the cluster (ie ports 80/443),

and then route the request to the correct service based on the url.

Cloud providers offer their own controller implementation options.

In my case, k3s [ships with traefik](https://docs.k3s.io/networking/networking-services) as its default ingress controller.

### Ingress

An [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) is a rule configuration read by the ingress controller.

![ingress](/deploy-expose/ingress.png)

I specify that traffic for the host `k3s.homelab.lan` should go to my homepage service.

### Applying the Manifests

With all my manifest yaml files saved, I use kubectl (aliased to 'k') to apply them.

![apply manifests](/deploy-expose/apply-manifests.png)

## Exposing the Service

With all resources created, the service is technically exposed outside the cluster.

However, in order to leverage the [SSL certs from ch 4](/ch-4-ssl-certs) across all my apps,

I'll route the traffic through the [reverse proxy from ch 5](/ch-5-reverse-proxy).

### Flow Diagram

SSL will be terminated at the reverse proxy, and http traffic forwarded to the cluster.

![traffic flow diagram](/deploy-expose/flow-diagram.png)

Unencrypted communication across my private network is fine for my homelab.

I'd never do this over a public network where the traffic could be intercepted!

### Nginx Config Update

To preserve services like the [registry from ch 6](/ch-6-image-registry) at `reg.homelab.lan`,

I add the following to the **bottom** of my nginx.conf:

![nginx config update](/deploy-expose/proxy-setup.png)

- leverage nginx as a load balancer to use both nodes in my cluster
- forward *.homelab.lan hosts (other than ones above in the file) to the load balanced servers

### DNS Update

Once again, I edit the `/etc/hosts` file on my homelab box,

and restart dnsmasq (see [Ch 3](/ch-3-dns)).

![dns update](/deploy-expose/etc-hosts-entry.png)

## Success!

![homepage in browser](/deploy-expose/homepage-on-browser.png)

It even includes metric widgets up top for RAM and CPU usage in the cluster!

(Which was achieved with the RBAC on the secret used by the homepage service account).

See [this commit](https://github.com/awgraves/homelab/commit/73d30a59983e3a89bb7d084a074faa516d3121ed) for the files from this chapter.
