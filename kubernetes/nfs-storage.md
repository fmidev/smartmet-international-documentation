# NFS Storage

Most Kubernetes applications need **persistent storage** — storage that survives when a pod is restarted or moved to another node. NFS (Network File System) is a simple and widely supported way to provide this.

There are two common ways to connect an NFS server to Kubernetes. Pick **one**.

| Option | When to use |
| --- | --- |
| [NFS Subdir External Provisioner](#option-1-nfs-subdir-external-provisioner) | Simple setup. Each volume becomes a subdirectory on the NFS server. Recommended for most cases. |
| [NFS CSI Driver](#option-2-nfs-csi-driver) | Newer standard. Use if you need features like volume snapshots or expansion. |

---

## Prerequisites

- An NFS server is running and reachable from every cluster node.
- The NFS export is configured to accept read/write access from the cluster nodes' IP range.
- You know the server address and the exported path.

---

## Option 1: NFS Subdir External Provisioner

This Helm chart installs a small pod that creates a new subdirectory on your NFS server for every `PersistentVolumeClaim`.

### Install

> [!IMPORTANT]
> Replace `nfs.server` and `nfs.path` with the address and exported path of **your** NFS server.

```bash
helm repo add nfs-subdir-external-provisioner \
  https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update

helm upgrade --install nfs-provisioner \
  nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --namespace nfs-provisioner \
  --create-namespace \
  --set nfs.server=172.16.3.89 \
  --set nfs.path=/smartmet/kubernetes_storage \
  --set storageClass.name=nfs-sc \
  --set storageClass.defaultClass=true
```

### Verify

```bash
kubectl get pods -n nfs-provisioner
kubectl get storageclass
```

The provisioner pod should show `Running` and `nfs-sc` should appear in the StorageClass list marked as `(default)`. After this, any `PersistentVolumeClaim` without an explicit `storageClassName` will use NFS automatically.

### Upgrade

```bash
helm repo update
helm upgrade --install nfs-provisioner \
  nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --namespace nfs-provisioner \
  --set nfs.server=172.16.3.89 \
  --set nfs.path=/smartmet/kubernetes_storage \
  --set storageClass.name=nfs-sc \
  --set storageClass.defaultClass=true
```

> `helm repo update` is important — without it, Helm uses its cached index and may not see the latest chart version.

---

## Option 2: NFS CSI Driver

The CSI (Container Storage Interface) driver is the modern replacement for in-tree NFS support.

### 1. Install the CSI driver

Follow the official installation guide: <https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/docs/install-csi-driver.md>

### 2. Create a StorageClass

> [!IMPORTANT]
> Adjust `server` and `share` in the file below to match your NFS server.

Create a file `sc-nfs.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 192.168.1.243
  share: /mnt/data/K8S
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=3
```

Apply it:

```bash
kubectl apply -f sc-nfs.yaml
```

### 3. Verify

```bash
kubectl get storageclass
kubectl logs -l app=csi-nfs-controller -n kube-system
```

`nfs-csi` should appear in the StorageClass list and the CSI controller logs should show no errors.

### Upgrade

Follow the [csi-driver-nfs upgrade instructions](https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/docs/install-csi-driver.md). The `StorageClass` itself rarely needs changes; if it does, edit `sc-nfs.yaml` and re-apply it with `kubectl apply -f sc-nfs.yaml`.
