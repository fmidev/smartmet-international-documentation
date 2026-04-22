# Geoweb

**Geoweb** is a web application for visualizing weather data, maps, and forecasts. It is published as a Helm chart in the FMI repository.

---

## Prerequisites

- [cert-manager](cert-manager.md) is installed with a `letsencrypt` ClusterIssuer (for HTTPS).
- An NGINX ingress controller is running in the cluster.
- The hostname you plan to use (for example `geoweb.example.com`) points (DNS) to your cluster's ingress IP.

## 1. Add the FMI Helm repository

```bash
helm repo add fmi https://fmidev.github.io/helm-charts
helm repo update
```

## 2. Create the values file

Create a file called `values.yaml`. Replace the hostname and adjust the Geoweb version if a newer one is available.

```yaml
versions:
  frontend: v12.0.0

frontend:
  url: geoweb.example.com
  env:
    GW_APP_URL: "https://geoweb.example.com"
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
        - geoweb.example.com
      secretName: geoweb-ingress-tls
  customAnnotations:
    cert-manager.io/cluster-issuer: letsencrypt
```

## 3. Install

```bash
helm upgrade --install geoweb fmi/geoweb-frontend \
  --namespace geoweb \
  --create-namespace \
  --values values.yaml
```

## 4. Verify

```bash
kubectl get pods -n geoweb
kubectl get ingress -n geoweb
```

When the certificate is ready, open `https://geoweb.example.com` in a browser.

## Upgrade

To install a newer version, edit the `versions.frontend` value in `values.yaml` and run the same `helm upgrade --install` command again.
