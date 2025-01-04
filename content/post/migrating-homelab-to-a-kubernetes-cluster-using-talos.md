+++
date = "2025-01-04T21:18:43+01:00"
title = "Migrating my homelab to a Kubernetes cluster using Talos"
draft = false
+++

After earning my Certified Kubernetes Application Developer (CKAD) certification, I wanted to bring Kubernetes to my homelab to maintain and expand my skills. Having previously followed Kelsey Hightower's ["Kubernetes the Hard Way"](https://github.com/kelseyhightower/kubernetes-the-hard-way), I sought an experience that would be simpler, more streamlined, and less error-prone. This led me to [Talos](https://talos.dev), an operating system purpose-built for running Kubernetes.

Talos offers a lightweight, secure, and fully automated way to deploy Kubernetes clusters. Designed with immutable infrastructure principles, Talos ensures that all management is API-driven, with no SSH access or package manager on the nodes, keeping the footprint minimal. This design makes Kubernetes clusters easier to install, manage, and upgrade.

![Talos dashboard](/images/talos-dashboard.png)
*Talos dashboard awaiting installation instructions.*

## The setup

In my homelab, I use two Dell OptiPlex microform computers running [Proxmox](https://proxmox.com/en/). To start off I created one control plane VM and two worker VMs booting using the Talos iso. To avoid opening any ports to the outside world, Talos includes built-in [WireGuard](https://www.wireguard.com/) functionality called [KubeSpan](https://www.talos.dev/v1.9/talos-guides/network/kubespan/), which seamlessly handles secure connectivity between nodes multi-cloud.

To enable external access, I’ve also configured a VPS on [Hetzner](https://hetzner.cloud/?ref=lzD1bYvAEQ8d) as a Talos Kubernetes worker node, accessing the three other nodes in my homelab through Wireguard. This provides an accessible entry point to my cluster from the outside world while ensuring my local environment remains secure.

![Homelab setup](/images/homelab.svg)
*Homelab setup with 4 Kubernetes nodes. Diagram created using [d2lang](https://d2lang.com/).*

## Installing Talos

The [Talos documentation](https://www.talos.dev/latest/) is much more comprehensive and clearer, so I highly recommend following it. My goal is to demonstrate how easy it is to install, use and upgrade a Kubernetes cluster. These steps are for installing Talos on the VMs running inside Proxmox, but it is also possible to run [a test cluster inside docker](https://www.talos.dev/v1.9/introduction/quickstart/).

1. **Download the Talos CLI**
   ```bash
   $ curl -Lo talosctl https://github.com/siderolabs/talos/releases/latest/download/talosctl-$(uname -s)-$(uname -m)
   $ chmod +x talosctl
   $ sudo mv talosctl /usr/local/bin/
   ```

2. **Generate cluster configuration**
   ```bash
   $ talosctl gen config my-cluster https://$CONTROL_PLANE_IP:6443
   ```
   This command generates the necessary configuration files (`worker.yaml` and `controlplane.yaml`) for your cluster.

3. **Install the control plane**  
   Boot your node(s) using the Talos image, then apply the configuration to the control plane node and bootstrap etcd:
   ```bash
   $ talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file controlplane.yaml
   $ export TALOSCONFIG="talosconfig"
   $ talosctl config endpoint $CONTROL_PLANE_IP
   $ talosctl config node $CONTROL_PLANE_IP
   $ talosctl bootstrap
   $ talosctl kubeconfig ~/.kube/config
   ```

4. **Add additional nodes**  
   Apply the appropriate configuration file (`worker.yaml` or `controlplane.yaml`) to the new node:
   ```bash
   $ talosctl apply-config --nodes <NODE_IP> --file <worker|controlplane>.yaml
   ```

![Talos dashboard](/images/talos-dashboard-installed.png)
*Talos dashboard after installation.*


## Upgrading
Upgrading Talos is straightforward and seamless. Use the following command to upgrade nodes:
```bash
$ talosctl upgrade --nodes <NODE_IP> --image ghcr.io/siderolabs/talos:<NEW_VERSION>
```

With Talos, upgrading Kubernetes is just as easy:
```bash
$ talosctl upgrade-k8s --nodes $CONTROL_PLANE_IP --version <NEW_K8S_VERSION>
```

## Migrating docker-compose files

If you’re transitioning from `docker-compose`, Talos supports the migration process with Kompose, a tool for converting docker compose files into Kubernetes manifests.

1. **Install Kompose**
   ```bash
   $ curl -L https://github.com/kubernetes/kompose/releases/download/v1.35.0/kompose-linux-amd64 -o kompose
   $ chmod +x kompose
   $ sudo mv kompose /usr/local/bin/
   ```

2. **Convert your docker compose file**
   ```bash
   $ kompose convert -f docker-compose.yaml -o application
   ```

This generates Kubernetes manifests that you can deploy to your Talos cluster.

## Essential components

I recommend these essential Kubernetes tools:

- **[Argo CD](https://argoproj.github.io/cd/)**: A GitOps continuous delivery tool for managing application deployments.
- **[ingress-nginx](https://kubernetes.github.io/ingress-nginx/)**: Handles HTTP and HTTPS traffic to services within your cluster.
- **[cert-manager](https://cert-manager.io/)**: Automates the management and issuance of TLS certificates.
- **[metallb](https://metallb.io/)**: Provides a load balancer implementation for bare-metal Kubernetes clusters.
- **[csi-driver-nfs](https://github.com/kubernetes-csi/csi-driver-nfs)**: Enables dynamic provisioning of storage using NFS.