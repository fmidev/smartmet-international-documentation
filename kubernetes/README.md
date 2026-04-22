# Kubernetes (RKE2) — Setup Guides

This folder contains step-by-step guides for installing and managing the applications that run on our Kubernetes clusters (RKE2).

Each guide is a single, self-contained page. Follow the guides in the order shown below when setting up a new cluster — some guides depend on earlier ones.

---

## 1. Before you start

| Guide | What it does |
| --- | --- |
| [Prepare Nodes](prepare-nodes.md) | Install `ipvsadm` and load IPVS kernel modules on every node. Must be done **before** installing Kubernetes. |
| [kubectl Basics](kubectl-basics.md) | Primer on the `kubectl` command-line tool used throughout these guides. Read this first if you are new to Kubernetes. |
| [Helm Basics](helm-basics.md) | Primer on the Helm package manager — every install guide below uses Helm. Read this first if you are new to Helm. |

> **New to Kubernetes?** This short video is a good overview before diving into the guides: <https://youtu.be/X48VuDVv0do>.

> This guide assumes Kubernetes (RKE2) is already installed after the nodes are prepared. See the [official RKE2 installation guide](https://docs.rke2.io/install/ha) for the cluster installation itself.

## 2. Core cluster services

Install these in order. They are the foundation that other applications rely on.

| Guide | What it does |
| --- | --- |
| [kube-vip](kube-vip.md) | Provides a single virtual IP address for the control plane. |
| [cert-manager](cert-manager.md) | Automatically issues and renews free TLS certificates from Let's Encrypt. |
| [NFS Storage](nfs-storage.md) | Connects the cluster to an NFS server for persistent storage. |

## 3. Cluster management

| Guide | What it does |
| --- | --- |
| [Portainer](portainer.md) | Web user interface for managing the cluster. |

## 4. Applications

| Guide | What it does |
| --- | --- |
| [Geoweb](geoweb.md) | Weather data visualization frontend (FMI Helm chart). |

## 5. Maintenance

| Guide | What it does |
| --- | --- |
| [Upgrade RKE2](rke2-upgrade.md) | Upgrade Kubernetes to a newer version. |

---

## Conventions used in these guides

- Commands in grey boxes are meant to be copied and run in a terminal on a machine that has `kubectl` and `helm` configured for the cluster.
- Text like `example.com`, `192.168.1.100`, or `your-email@example.com` is a **placeholder** — replace it with your own value before running the command.
- Coloured callouts highlight things that need attention:

  > [!IMPORTANT]
  > Values you **must change** before running the command (hostnames, IP addresses, email addresses, paths).

  > [!WARNING]
  > Steps where a mistake can cause real problems (time-sensitive actions, wrong order, risk of data loss).

  Plain blockquotes (without a `[!TYPE]` marker) are used for tips and extra information that is helpful but not critical.

## External references

- RKE2 documentation: <https://docs.rke2.io>
- Helm documentation: <https://helm.sh/docs>
- Kubernetes documentation: <https://kubernetes.io/docs>
