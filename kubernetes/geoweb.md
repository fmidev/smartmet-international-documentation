# Geoweb

**Geoweb** is a web application for visualizing weather data, maps, and forecasts. It is published as a Helm chart in the FMI repository.

Project: <https://gitlab.com/opengeoweb/opengeoweb>

---

## Prerequisites

- [cert-manager](cert-manager.md) is installed with a `letsencrypt` ClusterIssuer.
- An NGINX ingress controller is running in the cluster.
- A DNS record for the hostname you plan to use (for example `geoweb.example.com`) points to the cluster's ingress IP.

## 1. Add the FMI Helm repository

```bash
helm repo add fmi https://fmidev.github.io/helm-charts
helm repo update
```

## 2. Create the values file

> [!IMPORTANT]
> Replace `geoweb.example.com` with your real hostname — it appears three times below. Check the [FMI chart releases](https://github.com/fmidev/helm-charts/releases) and bump `versions.frontend` if a newer one is available.

Create a file called `geoweb-values.yaml`:

```yaml
versions:
  frontend: v12.0.0

frontend:
  url: geoweb.example.com
  env:
    GW_APP_URL: "https://geoweb.example.com"
    GW_FEATURE_APP_TITLE: "GeoWeb"
    GW_FEATURE_MENU_FE_VERSION: true
    GW_DEFAULT_THEME: "darkTheme"
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
  --values geoweb-values.yaml
```

## 4. Verify

```bash
kubectl get pods -n geoweb
kubectl get ingress -n geoweb
kubectl get certificate -n geoweb
```

The `Certificate` should eventually show `READY: True`. It can take a minute or two while cert-manager requests the certificate from Let's Encrypt.

When it is ready, open `https://geoweb.example.com` in a browser.

---

## Customise (optional)

Anything in this section is optional. Add the extra settings to `geoweb-values.yaml` and re-run the install command from step 3.

### Define map services and the initial view

You can pre-configure which WMS services appear in Geoweb and what map area is shown at startup.

> [!IMPORTANT]
> Replace the `bbox` values with coordinates that match your country, and replace the service URLs with real WMS endpoints you want to expose. `EPSG:3857` is the Web Mercator projection used by most web map services.

Add this inside `frontend:` in your values file:

```yaml
frontend:
  env:
    GW_INITIAL_PRESETS_FILENAME: custom/initialPresets.json

  customConfiguration:
    enabled: true
    files:
      "initialPresets.json":
        preset:
          presetType: "mapPreset"
          presetId: "mapPreset-1"
          presetName: "Default view"
          defaultMapSettings:
            proj:
              bbox:
                left: 3994293.3501
                bottom: 5596413.4629
                right: 5838565.9685
                top: 4567876.8103
              srs: "EPSG:3857"
          services:
            - name: "EUMETSAT"
              url: "https://view.eumetsat.int/geoserver/wms"
              id: "eumetsat"
            - name: "SmartMet"
              url: "https://data.example.com/wms"
              id: "smartmet"
```

### Require user login (Auth0 example)

Geoweb supports OAuth 2.0 with PKCE. The example below uses [Auth0](https://auth0.com), but any provider that supports the SPA + PKCE flow works — only the URLs in the env vars need to change.

**Step 1 — Create the Auth0 application**

1. Log in to the [Auth0 Dashboard](https://manage.auth0.com).
2. Go to **Applications → Create Application**.
3. Choose **Single Page Application**, give it a name, and click **Create**.
4. On the **Settings** tab, fill in:
   - **Allowed Callback URLs:** `https://geoweb.example.com/code`
   - **Allowed Logout URLs:** `https://geoweb.example.com`
   - **Allowed Web Origins:** `https://geoweb.example.com`
5. Click **Save Changes**.
6. Copy the **Domain** (for example `your-tenant.eu.auth0.com`) and the **Client ID** from the Settings tab.

> No client secret is needed — an SPA uses PKCE, which is a public flow and does not require a secret.

**Step 2 — Add the auth settings to `geoweb-values.yaml`**

> [!IMPORTANT]
> Replace `YOUR_AUTH0_DOMAIN` (three places) and `YOUR_AUTH0_CLIENT_ID` (once — only in `GW_AUTH_CLIENT_ID`) with the values you copied from Auth0. The `{client_id}`, `{app_url}`, `{state}`, and `{code_challenge}` tokens are template placeholders that Geoweb fills in at runtime — leave them as they are.

```yaml
frontend:
  env:
    GW_FEATURE_FORCE_AUTHENTICATION: true
    GW_AUTH_CLIENT_ID: "YOUR_AUTH0_CLIENT_ID"
    GW_AUTH_LOGIN_URL: "https://YOUR_AUTH0_DOMAIN/authorize?client_id={client_id}&response_type=code&scope=openid+profile+email&redirect_uri={app_url}/code&state={state}&code_challenge={code_challenge}&code_challenge_method=S256"
    GW_AUTH_TOKEN_URL: "https://YOUR_AUTH0_DOMAIN/oauth/token"
    GW_AUTH_LOGOUT_URL: "https://YOUR_AUTH0_DOMAIN/v2/logout?client_id={client_id}&returnTo={app_url}"
```

**Step 3 — Re-install and verify**

Run the install command from step 3 again, then confirm the auth config reached the pod:

```bash
curl -s https://geoweb.example.com/assets/config.json | grep GW_AUTH
```

You should see your Auth0 domain and client ID in the output. Visiting Geoweb in a browser should now redirect you to the Auth0 login page before loading the app.

---

## Upgrade

To install a newer Geoweb version, refresh the Helm repository, edit `versions.frontend` in `geoweb-values.yaml`, and reinstall:

```bash
helm repo update
helm upgrade --install geoweb fmi/geoweb-frontend \
  --namespace geoweb \
  --values geoweb-values.yaml
```

> `helm repo update` is important — without it, Helm uses its cached index and may not see the latest chart version.
