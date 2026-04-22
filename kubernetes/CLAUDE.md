# Conventions for `kubernetes/` documentation

Guides in this directory install and operate applications on the RKE2 Kubernetes cluster. The audience is operators and system administrators who are often new to Kubernetes, so guides must be **simple, self-contained, and copy-pasteable**.

Follow these conventions when adding or editing a guide.

## File layout

- One guide per application or Helm chart. File name is the app in lowercase with hyphens (`cert-manager.md`, `kube-vip.md`, `nfs-storage.md`).
- `README.md` is the index — guides are grouped by purpose (Before you start → Core services → Management → Applications → Maintenance) with a one-line description each.
- Flat structure: no subdirectories.
- When a new guide is added, add a row to the matching table in `README.md`.

## Standard page structure

Every app-install guide follows this order:

1. `# H1 title` — the app name as users see it.
2. One or two sentences describing what the app does and why a user would install it.
3. Documentation or project link on the next line, in angle-bracket form: `Documentation: <https://...>`.
4. `---` horizontal rule.
5. `## Prerequisites` — bullet list of things that must already exist (other charts installed, DNS records, ingress controller, etc.).
6. Numbered install sections: `## 1. ...`, `## 2. ...`. Typical steps: add Helm repo, create values file, install.
7. `## Verify` — commands to confirm the install worked, with the expected output or state noted in prose.
8. `## Customise (optional)` — optional extras, each as an `###` subsection. Only include when there are real customisations worth documenting.
9. `## Upgrade` — how to move to a newer version.

Guides that are not app installs (`prepare-nodes.md`, `rke2-upgrade.md`) may skip sections that do not apply.

## Helm conventions

- Use `helm upgrade --install` rather than `helm install`. It is idempotent, so the same command works for first install and later re-configuration.
- Include `helm repo add` + `helm repo update` inside the Install section so each guide is self-contained.
- Prefer a values file over long `--set` chains once the chart needs more than a handful of settings. Name it `{app}-values.yaml` (e.g. `portainer-values.yaml`, `geoweb-values.yaml`).
- The Upgrade section **must** run `helm repo update` before `helm upgrade --install`, with a reminder that Helm will otherwise use its cached index.

## Highlighting important bits

Use GitHub Alerts (they render with distinctive colour and an icon on GitHub) for the two cases below. Plain `>` blockquotes are still fine for tips and ordinary notes.

```markdown
> [!IMPORTANT]
> Values the reader must change before running the command
> (hostname, IP, email, path, client ID, token, ...).

> [!WARNING]
> Steps where a mistake causes real problems
> (time-sensitive actions, irreversible operations, risk of data loss).
```

Do **not** use emoji characters (`⚠️`, `❗`) for emphasis — the Alert icon is added automatically by GitHub and keeps the source clean.

## Placeholders

Use obvious placeholder values a reader can search for and replace:

| Type | Placeholder |
| --- | --- |
| Hostname | `example.com` (e.g. `geoweb.example.com`, `portainer.example.com`) |
| IP address | RFC1918 ranges such as `192.168.1.100` or `10.0.0.1` |
| Email | `your-email@example.com` |
| Token / secret / ID | ALL-CAPS with prefix, e.g. `YOUR_AUTH0_CLIENT_ID`, `YOUR_ECMWF_TOKEN` |

Never commit real production hostnames or credentials, even when a reference install is being adapted.

## Writing style

- Target audience is not deeply technical. Use short sentences and explain jargon on first use.
- Lead each guide with **what the tool does** in plain language before the install commands.
- Code blocks always have a language hint: ` ```bash `, ` ```yaml `, ` ```text `.
- Tables work well for comparing options (`When to use`) and explaining multiple settings in one place (`What the flag means`).
- URLs in prose use angle brackets: `<https://docs.rke2.io>`. Inline markdown links are fine for phrases (`[cert-manager](cert-manager.md)`).
- No emoji in prose. Bold is enough for emphasis in ordinary text.
