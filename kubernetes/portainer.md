# Portainer

**Portainer** is a web user interface for managing Kubernetes. It gives you a graphical overview of workloads, storage, logs, and more — useful when you do not want to use `kubectl` for every task.

Documentation: <https://docs.portainer.io/start/install-ce/server/kubernetes/baremetal>

---

## Prerequisites

- An NGINX ingress controller is running in the cluster.
- [cert-manager](cert-manager.md) is installed with a `letsencrypt` ClusterIssuer.
- A DNS record for the hostname you plan to use (for example `portainer.example.com`) points to the cluster's ingress IP.

## 1. Create the values file

Portainer has many ingress settings. A values file is much easier to read and edit than a long chain of `--set` flags.

> [!IMPORTANT]
> Replace `portainer.example.com` with your real hostname (it appears in three places).

Create a file called `portainer-values.yaml`:

```yaml
image:
  tag: lts

service:
  type: ClusterIP

tls:
  force: true

ingress:
  enabled: true
  ingressClassName: nginx
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: HTTPS
    cert-manager.io/cluster-issuer: letsencrypt
  hosts:
    - host: portainer.example.com
      paths:
        - path: /
  tls:
    - hosts:
        - portainer.example.com
      secretName: portainer-ingress-tls
```

### What the key settings mean

| Setting | Why it matters |
| --- | --- |
| `image.tag: lts` | Pin to the Long-Term Support release, which receives only stability fixes. |
| `service.type: ClusterIP` | Portainer is reached through the ingress only, not exposed directly on a node port. |
| `tls.force: true` | Portainer serves HTTPS internally, so the ingress must forward traffic as HTTPS (that is why the ingress annotation sets `backend-protocol: HTTPS`). |

## 2. Install

```bash
helm repo add portainer https://portainer.github.io/k8s/
helm repo update

helm upgrade --install portainer portainer/portainer \
  --namespace portainer \
  --create-namespace \
  --values portainer-values.yaml
```

## 3. First login

> [!WARNING]
> Create the admin account within 5 minutes of starting Portainer. For security reasons Portainer locks the initial setup if it is not used quickly. If you see a timeout error, restart the pod and open the browser again immediately:
>
> ```bash
> kubectl rollout restart deployment/portainer -n portainer
> ```

Open `https://portainer.example.com` in a browser and create your administrator account.

## 4. Verify

```bash
kubectl get pods -n portainer
kubectl get ingress -n portainer
kubectl get certificate -n portainer
```

The `Certificate` should eventually show `READY: True`. That confirms cert-manager obtained a TLS certificate from Let's Encrypt.

## Upgrade

To install a newer Portainer version, refresh the Helm repository first, then reinstall with the same values file:

```bash
helm repo update
helm upgrade --install portainer portainer/portainer \
  --namespace portainer \
  --values portainer-values.yaml
```

> `helm repo update` is important — without it, Helm uses its cached index and may not see the latest chart version.
