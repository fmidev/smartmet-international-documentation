# Helm Basics

**Helm** is the package manager for Kubernetes. Instead of writing Kubernetes YAML by hand, Helm installs applications from pre-built packages called **charts**. Every other install guide in this directory uses Helm to install something, so it pays to understand the basics first.

Documentation: <https://helm.sh/docs>

---

## Key concepts

| Term | What it means |
| --- | --- |
| **Chart** | A packaged application — everything Kubernetes needs to run it, bundled together (similar to a `.deb` or `.rpm` package). |
| **Repository** | A collection of charts hosted online. You add a repo once, then install charts from it. |
| **Release** | A running instance of a chart in your cluster. The same chart can be installed several times as different releases, under different names. |
| **Values** | Configuration options you pass to a chart to customise the release. |

## Install Helm

Install Helm on the machine where you run `kubectl` (not on the cluster nodes). On Rocky Linux / RHEL:

```bash
sudo dnf install helm
```

For other platforms see <https://helm.sh/docs/intro/install>.

Check it works:

```bash
helm version
```

## Add a chart repository

Before installing a chart, add the repository it lives in:

```bash
helm repo add REPO_NAME REPO_URL
```

List the repositories you have added:

```bash
helm repo list
```

Refresh the local index so Helm sees newly published chart versions:

```bash
helm repo update
```

> [!IMPORTANT]
> Always run `helm repo update` before upgrading a release. Helm caches the chart index locally — without updating, it will install whatever version is in the cache, not the latest available.

## Find a chart

Search the repositories you have added:

```bash
helm search repo CHART_NAME
```

List every available version of a chart (useful when pinning to a specific version):

```bash
helm search repo CHART_NAME --versions
```

## Namespaces — always pick one

A **namespace** is Kubernetes' way of grouping related resources (pods, services, volumes, secrets, …). Every resource lives in exactly one namespace. If you do not name one, Helm installs the release into the `default` namespace.

> [!IMPORTANT]
> Never install a release into the `default` namespace. Always pass `--namespace NAMESPACE --create-namespace` to every `helm install` / `helm upgrade` command.
>
> Reasons:
> - Apps in `default` mix with each other — cleanup, backups, and access control become hard.
> - Two charts that use the same resource names will collide.
> - Monitoring, quotas, and RBAC are almost always set up per namespace.
> - Moving a release to a different namespace later requires uninstalling and reinstalling it (you can't simply rename it).
>
> The convention in these guides is to use the app name as the namespace — `portainer` for Portainer, `cert-manager` for cert-manager, and so on.

## Install or upgrade a chart

Use `helm upgrade --install` for both installing and upgrading. It installs the release if it does not exist, and upgrades it if it does — so the same command is safe to re-run after fixing a mistake or changing values.

With a values file (the usual pattern in these guides):

```bash
helm upgrade --install RELEASE_NAME REPO_NAME/CHART_NAME \
  --namespace NAMESPACE \
  --create-namespace \
  --values values.yaml
```

With inline `--set` flags for simple overrides:

```bash
helm upgrade --install RELEASE_NAME REPO_NAME/CHART_NAME \
  --namespace NAMESPACE \
  --create-namespace \
  --set key=value
```

> A **values file** is a YAML file that holds your custom configuration. The other guides in this directory use files like `portainer-values.yaml` and `geoweb-values.yaml` — each is a values file for that chart.

## Check what is installed

List all releases in every namespace:

```bash
helm list -A
```

Show the detailed status of one release:

```bash
helm status RELEASE_NAME --namespace NAMESPACE
```

## Inspect a chart before installing

See all configuration options a chart supports, with their default values:

```bash
helm show values REPO_NAME/CHART_NAME
```

Save them to a file you can edit as a starting point:

```bash
helm show values REPO_NAME/CHART_NAME > my-values.yaml
```

## Uninstall a release

> [!WARNING]
> `helm uninstall` removes every Kubernetes resource that belongs to the release. If the release owns a `PersistentVolumeClaim` that holds real data, that data may be deleted too. Back up anything important first.

```bash
helm uninstall RELEASE_NAME --namespace NAMESPACE
```

## Quick reference

| Task | Command |
| --- | --- |
| Install Helm (Rocky / RHEL) | `sudo dnf install helm` |
| Add a repository | `helm repo add REPO_NAME REPO_URL` |
| Refresh repository index | `helm repo update` |
| Search for a chart | `helm search repo CHART_NAME` |
| See available chart versions | `helm search repo CHART_NAME --versions` |
| Install or upgrade a release | `helm upgrade --install RELEASE REPO/CHART --values values.yaml -n NS --create-namespace` |
| List installed releases | `helm list -A` |
| Show a release's status | `helm status RELEASE -n NS` |
| Show default chart values | `helm show values REPO/CHART` |
| Uninstall a release | `helm uninstall RELEASE -n NS` |
