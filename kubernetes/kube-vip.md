# kube-vip

**kube-vip** provides a single virtual IP address (VIP) for the Kubernetes control plane. Clients connect to this VIP instead of a specific node, so the cluster stays reachable even if one control plane node goes down.

Documentation: <https://kube-vip.io>

---

## Prerequisites

kube-vip relies on the IPVS kernel modules for load balancing. Before installing it, make sure every node has IPVS set up — see [Prepare Nodes](prepare-nodes.md). Without IPVS, kube-vip will fail to balance traffic and the VIP will not work correctly.

## Install

> [!IMPORTANT]
> Replace `192.168.1.100` with the VIP address you want to use. It must be a **free IP** on the same subnet as your cluster nodes.

```bash
helm repo add kube-vip https://kube-vip.github.io/helm-charts
helm repo update

helm upgrade --install kube-vip kube-vip/kube-vip \
  --namespace kube-system \
  --set config.address=192.168.1.100 \
  --set env.cp_enable=true \
  --set env.vip_leaderelection=true \
  --set env.svc_election=true
```

## What the flags mean

| Flag | Purpose |
| --- | --- |
| `config.address` | The virtual IP address assigned to the control plane. |
| `env.cp_enable` | Enable load balancing for the control plane. |
| `env.vip_leaderelection` | Elect a leader node to hold the VIP (required for Layer 2 mode). |
| `env.svc_election` | Allow kube-vip to also serve VIPs for Kubernetes Services. |

All other settings keep their default values.

## Verify

Check that the kube-vip pods are running on the control plane nodes:

```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=kube-vip
```

Every pod should show `Running`. Then confirm the VIP is reachable from another machine on the same network:

```bash
ping 192.168.1.100
```

## Upgrade

Refresh the Helm repository and re-run the install command with the same settings:

```bash
helm repo update
helm upgrade --install kube-vip kube-vip/kube-vip \
  --namespace kube-system \
  --set config.address=192.168.1.100 \
  --set env.cp_enable=true \
  --set env.vip_leaderelection=true \
  --set env.svc_election=true
```

> `helm repo update` is important — without it, Helm uses its cached index and may not see the latest chart version.
