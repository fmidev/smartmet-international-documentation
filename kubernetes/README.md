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
  lb_port: "6443" 
  vip_subnet: "24"
  cp_enable: "true" # enable control plane load balancing
  svc_enable: "true" # enable user plane load balancing
  vip_leaderelection: "true" # mandatory for L2 mode
  svc_election: "true"
```


```
helm repo add kube-vip https://kube-vip.github.io/helm-charts --force-update
```

```
helm upgrade --install \
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
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.17.0 \
  --set crds.enabled=true
```

### Install FMI helm repository 
```
helm repo add fmi https://fmidev.github.io/helm-charts --force-update
```
### Install portainer
Documentation: https://docs.portainer.io/start/install-ce/server/kubernetes/baremetal

```
helm repo add portainer https://portainer.github.io/k8s/ --force-update
```

```
helm upgrade --install \
  portainer portainer/portainer \
  --namespace portainer \
  --create-namespace \
  --set service.type=ClusterIP \
  --set tls.force=true \
  --set image.tag=lts \
  --set ingress.enabled=true \
  --set ingress.ingressClassName=nginx \
  --set ingress.annotations."nginx\.ingress\.kubernetes\.io/backend-protocol"=HTTPS \
  --set ingress.annotations."cert-manager\.io/cluster-issuer"=letsencrypt \
  --set ingress.hosts\[0\].host=portainer.some.domain \
  --set ingress.hosts\[0\].paths\[0\].path="/" \
  --set ingress.tls\[0\].hosts\[0\]=portainer.some.domain \
  --set ingress.tls\[0\].secretName=portainer-ingress-tls
```

## UPGRADE

Upgrade RKE2 
```
curl -sfL https://get.rke2.io | sh -
```

### Install NFS external provisioner
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=172.16.3.89 \
  --set nfs.path=/smartmet/kubernetes_storage \
  --create namespace \
  --namespace nfs-provisioner
  --set storageClass.name=nfs-sc \
  --set storageClass.defaultClass=true
```
```
kubectl apply -f sc_nfs.yaml
```

### Install NFS external provider
```
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
```
kubectl apply -f sc_nfs.yaml -n sc
kubectl logs -l app=nfs-client-provisioner -n <namespace>
```

### Install Letsencrypt cert manager
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: smartmet@weatherproof.fi
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      # This is your identity with your ACME provider. Any secret name
      # may be chosen. It will be populated with data automatically,
      # so generally nothing further needs to be done with
      # the secret. If you lose this identity/secret, you will be able to
      # generate a new one and generate certificates for any/all domains
      # managed using your previous account, but you will be unable to revoke
      # any certificates generated using that previous account.
      name: example-issuer-account-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```
```
kubectl apply -f letsencrypt.yaml -n cert-manager
```

### Install Geoweb
```
versions:
  frontend: v12.0.0

frontend:
  url: geoweb.meteo.ge
  env:
    GW_APP_URL: "https://geoweb.meteo.ge"
    GW_FEATURE_MENU_FE_VERSION: true
    GW_DEFAULT_THEME: "darkTheme"
    GW_FEATURE_APP_TITLE: "GeoWeb"
  resources:
      requests:
        memory: "100Mi"
        cpu: "100m"
      limits:
        memory: "2Gi"
        cpu: "1"

ingress:
  tls:
    - hosts:
        - geoweb.meteo.ge
      secretName: geoweb-ingress-tls

  customAnnotations:
    cert-manager.io/cluster-issuer: letsencrypt
```
```
helm upgrade --install geoweb fmi/geoweb-frontend --namespace geoweb --create-namespace --values=values.yaml
```
