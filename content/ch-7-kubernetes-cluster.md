---
weight: 1
title: "Ch 7: Kubernetes Cluster"
bookToc: true
type: "docs"
bookFlatSection: true
---

# Kubernetes Cluster

The time has come!

My homelab ecosystem will grow to include a kubernetes cluster. 💪

## Additional Hardware

Sure, I could use virtualization to simulate more machines.

But I think its cooler to have the real thing.

<img src="/cluster/cluster_hardware.jpg" style="max-width: 400px"/>

Additions:
- 1 Netgear 5 Port Gigabit Network Switch ($25)
- 2 HP Prodesk 400 G3 Mini PCs off Ebay ($170 for the pair)
  - Pentium G4400T CPU (2.9Ghz)
  - 8 GB RAM
  - 256 GB SSD
- 3 Cat6 cables to wire it all together ($15)

Total homelab hardware cost now sits at $310.

### Roles

I'll leave my 'homelab' host dedicated to all non-kubernetes services.

The 2 HPs will become the cluster.

This maintains a clean separation of concerns.

### OS Installation

I repeat similar steps in [Ch 1](/ch-1-linux-box) to install Debian on both new machines.

<img src="/cluster/debian_installs.jpg" style="max-width: 400px"/>

They are given the host names 'control-plane' and 'worker1' respectively.

### Network Setup

The DHCP IP reservations are added, just like I did for my main box.

![IP reservations](/cluster/updated_ip_reservations.png)

### SSH

Same as in [Ch 2](/ch-2-ssh), I set up my SSH keys and update my laptop's ssh config:

![ssh config update](/cluster/ssh_config_update.png)

## Kubernetes

Many [distributions of kubernetes](https://www.glukhov.org/post/2025/08/kubernetes-distributions-overview/#-major-kubernetes-variants-for-self-hosting) are available for different uses.

While I do initially consider full-blown k8s with [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/),

I decide on [k3s](https://k3s.io/) instead for a few reasons:
- single binary install
- easier to maintain / upgrade
- not too different from the convenience of managed cloud clusters

The architecture does diverge somewhat from k8s, but I'm ok with that.

I'll follow along with the [quick start documentation](https://docs.k3s.io/quick-start).

### Control Plane

I start with the installation on the control-plane host and verify the k3s service is running:

![k3s running](/cluster/k3s_running.png)

### Worker Node

I copy-paste the token value from the control-plane host into a .txt file on the worker1 host.

Then I run a modified install command using the IP of my control-plane host and the join token value.

![worker install](/cluster/worker_install.png)

### Kubectl

From my control-plane host, I verify the cluster is up.

![cluster ready](/cluster/cluster_ready.png)

I copy the contents of `/etc/rancher/k3s/k3s.yaml` file from the control-plane to `~/.kube/config` on both my homelab and laptop hosts.

Now I can access my cluster externally from those machines:

![external cluster access](/cluster/external_access.png)

Awesome!
