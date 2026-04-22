# Portainer

**Portainer** is a web user interface for managing Kubernetes. It gives you a graphical overview of workloads, storage, logs, and more — useful when you do not want to use `kubectl` for every task.

Documentation: <https://docs.portainer.io/start/install-ce/server/kubernetes/baremetal>

---

## Prerequisites

- An NGINX ingress controller is running in the cluster.
- [cert-manager](cert-manager.md) is installed with a `letsencrypt` ClusterIssuer.

## Install

> Replace `portainer.example.com` with the real hostname you want to use. The hostname must point (DNS) to your cluster's ingress IP.

```bash
helm repo add portainer https://portainer.github.io/k8s/
helm repo update

helm upgrade --install portainer portainer/portainer \
  --namespace portainer \
  --create-namespace \
  --set service.type=ClusterIP \
  --set tls.force=true \
  --set image.tag=lts \
  --set ingress.enabled=true \
  --set ingress.ingressClassName=nginx \
  --set ingress.annotations."nginx\.ingress\.kubernetes\.io/backend-protocol"=HTTPS \
  --set ingress.annotations."cert-manager\.io/cluster-issuer"=letsencrypt \
  --set ingress.hosts\[0\].host=portainer.example.com \
  --set ingress.hosts\[0\].paths\[0\].path="/" \
  --set ingress.tls\[0\].hosts\[0\]=portainer.example.com \
  --set ingress.tls\[0\].secretName=portainer-ingress-tls
```

## First login

Open `https://portainer.example.com` in a browser. The first time you open it, Portainer will ask you to create an administrator account.

> You must open Portainer within a few minutes of installation. For security reasons, Portainer disables the initial setup if it is not used quickly. If that happens, restart the Portainer pod:
>
> ```bash
> kubectl rollout restart deployment/portainer -n portainer
> ```
