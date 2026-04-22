# Prepare Nodes

Before installing Kubernetes, every Linux node must be prepared with **IPVS** (IP Virtual Server). IPVS is used by Kubernetes to route network traffic between nodes. It is faster and more reliable than the older `iptables` method.

> Run the steps below on **every node** of the cluster (control plane and workers).

---

## 1. Install `ipvsadm`

`ipvsadm` is the tool used to manage IPVS rules.

```bash
sudo yum install -y ipvsadm
```

## 2. Load the IPVS kernel modules

Create the file `/etc/modules-load.d/ipvs.conf` with the following content:

```text
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
```

Load the modules immediately (no reboot needed):

```bash
sudo systemctl restart systemd-modules-load.service
```

## 3. Verify

Confirm that the modules are loaded:

```bash
lsmod | grep ip_vs
```

You should see `ip_vs`, `ip_vs_rr`, `ip_vs_wrr`, `ip_vs_sh`, and `nf_conntrack` listed.
