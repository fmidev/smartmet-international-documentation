# SmartMet - Kubernetes RKE2

## Installation

### Prepare the nodes

#### Install ipvsadm
This tool is required for configuring load balancing using IPVS (IP Virtual Server), which is more efficient for handling traffic across Kubernetes nodes.
```
sudo yum install -y ipvsadm
```
Edit `/etc/modules-load.d/ipvs.conf`
```
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
```

Load IPVS modules without rebooting 
```
sudo systemctl restart systemd-modules-load.service
```


### Install kube-vip
Documentation: https://kube-vip.io

Create file `kube-vip.cnf`. Adjust address and vip_interface accordingly.
```
config:
  address: "192.168.1.100"
env:
  vip_interface: "eno1" # This is the interface, same on all nodes where the VIP is announced
  vip_arp: "true"  # mandatory for L2 mode
  lb_enable: "true"
  lb_port: "16443" # changed as microk8s uses 16443 instead of 6443
  vip_cidr: "24"
  cp_enable: "true" # enable control plane load balancing
  svc_enable: "true" # enable user plane load balancing
  vip_leaderelection: "true" # mandatory for L2 mode
  svc_election: "true"
```


```
helm repo add kube-vip https://kube-vip.github.io/helm-charts --force-update
```

```
helm install \
  kube-vip kube-vip/kube-vip \
  --namespace kube-system \
  --create-namespace \
  -f kube-vip.cnf
```

### Install cert-manager
Documentation: https://cert-manager.io/docs/installation/helm/
```
helm repo add jetstack https://charts.jetstack.io --force-update
```

Adjust version to the latest available.
```
microk8s helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.17.0 \
  --set crds.enabled=true
```


