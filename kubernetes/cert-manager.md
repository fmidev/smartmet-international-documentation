# cert-manager

**cert-manager** automatically requests and renews TLS certificates (for HTTPS) from Let's Encrypt. Once installed, any Ingress in the cluster can get a free, valid certificate by adding a single annotation.

Documentation: <https://cert-manager.io/docs/installation/helm/>

---

## 1. Install cert-manager

> Check <https://github.com/cert-manager/cert-manager/releases> for the latest version and replace `v1.17.0` below if a newer one is available.

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.17.0 \
  --set crds.enabled=true
```

Verify the pods are running:

```bash
kubectl get pods -n cert-manager
```

## 2. Create a Let's Encrypt ClusterIssuer

A **ClusterIssuer** tells cert-manager where to request certificates from. One issuer is enough for the whole cluster.

> [!IMPORTANT]
> Before you apply this file, change the `email:` value to a real address you control. Let's Encrypt uses it to warn you about expiring certificates and to contact you about account issues. A fake or unreachable address can cause the issuer to be rejected.

Create a file called `letsencrypt.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    # IMPORTANT: Replace with your own email.
    # Let's Encrypt uses it to notify you about expiring certificates.
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-account-key
    solvers:
      - http01:
          ingress:
            ingressClassName: nginx
```

Apply it:

```bash
kubectl apply -f letsencrypt.yaml
```

## 3. Use the issuer in an Ingress

To get a certificate automatically, add this annotation to any Ingress:

```yaml
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
```

cert-manager will request a certificate from Let's Encrypt and store it in the `Secret` named in `tls.secretName`.
