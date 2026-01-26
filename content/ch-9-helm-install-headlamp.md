---
weight: 1
title: "Ch 9: Helm Install Headlamp"
bookToc: true
type: "docs"
bookFlatSection: true
---

# Helm Install Headlamp

[Last chapter](/ch-8-deploy-expose), I deployed an app by manually creating & applying manifest files.

This time I'll use Helm as a more convenient way to install my next app: Headlamp.

## Helm

[Helm](https://helm.sh/) is "the package manager for kubernetes".

It allows developers to bundle together all the related manifest files for a given app(s).

This makes it easier to install, configure, and upgrade applications on a cluster.

### Installation

Easily done - I follow [the guide](https://helm.sh/docs/intro/install) and use the install script.

![helm installed](/helm-headlamp/install_helm.png)

## Headlamp

[Headlamp](https://headlamp.dev/) is the successor to the (now deprecated) [kubernetes dashboard](https://github.com/kubernetes/dashboard?tab=readme-ov-file#important).

Its a UI that lets you visualize and manage your cluster resources via a web interface.

### Helm Install

Following the [helm chart](https://headlamp.dev/docs/latest/installation/in-cluster/#using-helm), I run the following:

```bash
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
helm install my-headlamp headlamp/headlamp -n kube-system
```

![headlamp installation](/helm-headlamp/headlamp_helm.png)

This adds the headlamp repo and installs it in the kube-system namespace.

![headlamp resources](/helm-headlamp/headlamp_resources.png)

### Ingress

I'll still need to set up my own ingress to expose the installed app.

I create a new manifest file for one in my homelab repo at `/kubernetes/headlamp/headlamp-ingress.yaml`:

![headlamp ingress](/helm-headlamp/headlamp_ingress.png)

I choose to serve this at `headlamp-k3s.homelab.lan`

And apply the manifest with kubectl:

![headlamp ingress applied](/helm-headlamp/headlamp_ingress_applied.png)

### DNS Entry

As always, I add an entry for this in my /etc/hosts file on the homelab box:

(see [ch 8](/ch-8-deploy-expose) where I configured a wildcard proxy for subdomains to my cluster).

![hosts file update](/helm-headlamp/hosts_update.png)

And restart dnsmasq with systemctl (see [ch 3 DNS](/ch-3-dns)).

### Browser Access

Now when I visit `https://headlamp-k3s.homelab.lan` on my local network, I see this!

![headlamp login](/helm-headlamp/headlamp_login.png)

### Access Token

The [headlamp docs recommend](https://headlamp.dev/docs/latest/installation/#create-a-service-account-token) creating an admin service account to use for login.

I specify the SA and ClusterRoleBinding in a new `admin-service-account.yaml`:

![admin service account manifest](/helm-headlamp/admin_service_account.png)

Apply it, and get a token to use:

![getting headlamp access token](/helm-headlamp/get_admin_token.png)

After pasting in the token value, I'm successfully logged in to headlamp!

![headlamp logged in](/helm-headlamp/headlamp_logged_in.png)

## Homepage Updates

I want to easily access headlamp from the homepage I deployed in [ch 8](/ch-8-deploy-expose).

### Annotations for Headlamp

The homepage app ingress manifest had annotations that populated it on the UI.

I edit my `headlamp-ingress.yaml` file to include similar ones:

![updated ingress annotations](/helm-headlamp/update_ingress_annotations.png)


After reapplying the headlamp-ingress and visiting `https://k3s.homelab.lan`, 

behold, it works!

![headlamp on homepage](/helm-headlamp/headlamp_added_to_homepage.png)

The 'headlamp.png' is an [included icon](https://gethomepage.dev/configs/services/#icons) for the homepage app.

### Remove Placeholder Services

Now seems like a good time for a quick clean up.

I edit the homepage ConfigMap and replace the `services.yaml` values with an empty string.

![configmap cleanup](/helm-headlamp/config_map_cleanup.png)

I also remove the annotations from the homepage Ingress,

as I don't need a link ***to*** the homepage ***on*** the homepage.

![homage ingress cleanup](/helm-headlamp/homepage_ingress_cleanup.png)

### Clean Homepage

Now I can access Headlamp directly from a newly cleaned up landing page. 👍

![cleaned homepage](/helm-headlamp/clean_homepage.png)

See [this commit](https://github.com/awgraves/homelab/commit/8a2cef4296ba4a90d23e18bd5889ac2ff3f49d57) with the files from this chapter.
