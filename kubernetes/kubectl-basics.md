# kubectl Basics

**kubectl** is the command-line tool for Kubernetes. It is how you talk to the cluster from your terminal — listing pods, reading logs, applying YAML manifests, restarting deployments, and more. Every guide in this directory uses kubectl at some point.

Documentation: <https://kubernetes.io/docs/reference/kubectl/>

---

## Key concepts

| Term | What it means |
| --- | --- |
| **Resource** | Anything Kubernetes manages — pods, services, deployments, secrets, ingresses, and so on. |
| **Manifest** | A YAML (or JSON) file that describes a resource. You send it to the cluster with `kubectl apply -f FILE.yaml`. |
| **Namespace** | A group of related resources (see below). |
| **Kubeconfig** | A file that tells kubectl *which* cluster to talk to and *who* you are. Default location: `~/.kube/config`. |
| **Context** | A named pair of (cluster, user, namespace) inside a kubeconfig. Switching context switches which cluster you work with. |

## Install kubectl

Install kubectl on the machine from which you will run cluster commands. RKE2 cluster nodes already ship a kubectl at `/var/lib/rancher/rke2/bin/kubectl`.

Follow the official installation guide for your platform: <https://kubernetes.io/docs/tasks/tools/#kubectl>

Check it works:

```bash
kubectl version --client
```

## Configure kubectl for RKE2

kubectl reads cluster details from a **kubeconfig** file, by default `~/.kube/config`.

### From a control plane node

RKE2 ships kubectl and a kubeconfig on each control plane node. Add them to your shell environment:

```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin

kubectl get nodes
```

Add those `export` lines to `~/.bashrc` to make them permanent.

### From another machine

Copy the kubeconfig from a control plane node:

```bash
mkdir -p ~/.kube
scp root@CONTROL_PLANE_IP:/etc/rancher/rke2/rke2.yaml ~/.kube/config
```

> [!IMPORTANT]
> The copied file has `server: https://127.0.0.1:6443`, which only works from the control plane node itself. Edit `~/.kube/config` and change that line to point at your kube-vip VIP:
>
> ```yaml
> server: https://YOUR_VIP:6443
> ```

Test:

```bash
kubectl get nodes
```

## Namespaces — always specify one

A **namespace** groups related resources. Most kubectl commands operate on the current namespace, which is `default` unless you say otherwise.

> [!IMPORTANT]
> Always pass `--namespace NAMESPACE` (short form `-n NAMESPACE`). Without it, kubectl looks in the `default` namespace — where your workloads should not live (see [Helm Basics](helm-basics.md)).
>
> Tip: set a default namespace for the current kubeconfig context, so you can skip `-n` for most commands in this session:
>
> ```bash
> kubectl config set-context --current --namespace=NAMESPACE
> ```

List all namespaces in the cluster:

```bash
kubectl get namespaces
```

## List resources

Pods in one namespace:

```bash
kubectl get pods -n NAMESPACE
```

Pods in every namespace:

```bash
kubectl get pods -A
```

Extra columns such as the pod IP and the node it runs on:

```bash
kubectl get pods -n NAMESPACE -o wide
```

Show a resource as YAML (useful for learning or debugging):

```bash
kubectl get pod POD_NAME -n NAMESPACE -o yaml
```

Other common resource types: `deployments`, `services`, `ingresses`, `secrets`, `configmaps`, `pvc` (persistent volume claims), `nodes`, `storageclass`, `certificate` (from cert-manager).

## Describe a resource

`describe` shows the full details of a resource, including the recent events the cluster recorded for it. This is usually the first command to run when something is broken.

```bash
kubectl describe pod POD_NAME -n NAMESPACE
kubectl describe ingress INGRESS_NAME -n NAMESPACE
kubectl describe certificate CERT_NAME -n NAMESPACE
```

## Read logs

Show a pod's logs:

```bash
kubectl logs POD_NAME -n NAMESPACE
```

Follow logs in real time (like `tail -f`):

```bash
kubectl logs -f POD_NAME -n NAMESPACE
```

If a pod has multiple containers, pick one with `-c CONTAINER_NAME`.

The previous container's logs, for example after a crash:

```bash
kubectl logs POD_NAME -n NAMESPACE --previous
```

## Run a command inside a pod

Open an interactive shell:

```bash
kubectl exec -it POD_NAME -n NAMESPACE -- /bin/sh
```

If the container has `bash`, you can use `/bin/bash` instead.

Run a single command without a shell:

```bash
kubectl exec POD_NAME -n NAMESPACE -- ls /var/log
```

## Apply and delete manifests

Apply a YAML file (creates the resource if it is missing, updates it if it already exists):

```bash
kubectl apply -f FILE.yaml
```

Apply every YAML in a directory:

```bash
kubectl apply -f ./manifests/
```

> [!WARNING]
> `kubectl delete` permanently removes a resource. For resources that own data (for example `PersistentVolumeClaim`), the underlying data may also be deleted. Check the name and namespace before running the command.

Delete the resources described by a file:

```bash
kubectl delete -f FILE.yaml
```

Delete a specific resource by name:

```bash
kubectl delete pod POD_NAME -n NAMESPACE
kubectl delete deployment DEPLOYMENT_NAME -n NAMESPACE
```

## Restart a deployment

Rolling restart of every pod in a deployment — the clean way, with no downtime if the deployment has more than one replica:

```bash
kubectl rollout restart deployment/DEPLOYMENT_NAME -n NAMESPACE
```

Check the status of a rollout:

```bash
kubectl rollout status deployment/DEPLOYMENT_NAME -n NAMESPACE
```

## Port forwarding

Open a local port that tunnels into a service or pod inside the cluster. Useful for reaching an internal service without exposing it through an ingress.

```bash
kubectl port-forward -n NAMESPACE service/SERVICE_NAME LOCAL_PORT:REMOTE_PORT
```

For example, reach the Portainer service on `https://localhost:9443`:

```bash
kubectl port-forward -n portainer service/portainer 9443:9443
```

Leave the command running for as long as you need the tunnel. Stop it with `Ctrl+C`.

## Useful tips

- **Check the current cluster before destructive commands.** `kubectl config current-context` shows which cluster you are talking to right now.
- **Multiple clusters** — `kubectl config get-contexts` lists them, `kubectl config use-context NAME` switches.
- **Shell completion** — bash: `source <(kubectl completion bash)`; zsh: `source <(kubectl completion zsh)`. Add the line to `~/.bashrc` or `~/.zshrc` to make it permanent. On Rocky Linux you may need `sudo dnf install bash-completion` first.
- **Short alias** — many operators add `alias k=kubectl` to their shell profile to save typing.

## Quick reference

| Task | Command |
| --- | --- |
| List pods in a namespace | `kubectl get pods -n NS` |
| List pods everywhere | `kubectl get pods -A` |
| Describe a resource | `kubectl describe TYPE NAME -n NS` |
| Read logs | `kubectl logs POD -n NS` |
| Follow logs | `kubectl logs -f POD -n NS` |
| Open a shell in a pod | `kubectl exec -it POD -n NS -- /bin/sh` |
| Apply a manifest | `kubectl apply -f FILE.yaml` |
| Delete a resource | `kubectl delete TYPE NAME -n NS` |
| Restart a deployment | `kubectl rollout restart deployment/NAME -n NS` |
| Port-forward | `kubectl port-forward -n NS service/NAME LOCAL:REMOTE` |
| Set default namespace | `kubectl config set-context --current --namespace=NS` |
| Current cluster | `kubectl config current-context` |
| Switch cluster | `kubectl config use-context NAME` |
