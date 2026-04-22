# Upgrade RKE2

RKE2 is the Kubernetes distribution used for this cluster. To upgrade to the latest stable version, run the installation script again on **each node**.

Official upgrade guide: <https://docs.rke2.io/upgrade/basic_upgrade>

---

## 1. Back up first

Before upgrading, make sure you have a recent backup of the cluster's etcd database. See <https://docs.rke2.io/backup_restore>.

## 2. Upgrade the control plane nodes

Upgrade one control plane node at a time. On each node, run:

```bash
curl -sfL https://get.rke2.io | sh -
sudo systemctl restart rke2-server
```

Wait until the node is `Ready` again before moving to the next one:

```bash
kubectl get nodes
```

## 3. Upgrade the worker nodes

Repeat the same procedure on every worker node, but restart the agent service instead:

```bash
curl -sfL https://get.rke2.io | sh -
sudo systemctl restart rke2-agent
```

## 4. Verify

Check that all nodes report the new version:

```bash
kubectl get nodes -o wide
```
