# home-assistant

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/helm-home-assistant)](https://artifacthub.io/packages/search?repo=helm-home-assistant)
![Version: 1.1.2](https://img.shields.io/badge/Version-1.1.2-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 2026.5.1](https://img.shields.io/badge/AppVersion-2026.5.1-informational?style=flat-square)

Home Assistant on Kubernetes with optional code-server addon and Authentik outpost

This chart deploys [Home Assistant](https://www.home-assistant.io) as a `StatefulSet` with persistent
storage for `/config`. It ships several opt-in addons and integrations that are disabled by default —
enable only what your cluster already provides.

## Usage

**Helm repo (GitHub Pages):**

```bash
helm repo add sironite https://sironite.github.io/helm-home-assistant
helm repo update
helm install home-assistant sironite/home-assistant \
  --namespace home-assistant --create-namespace \
  -f my-values.yaml
```

**OCI (GHCR):**

```bash
helm install home-assistant oci://ghcr.io/sironite/home-assistant \
  --version 1.1.2 \
  --namespace home-assistant --create-namespace \
  -f my-values.yaml
```

## Addons and integrations

### code-server (`addons.codeserver`)

Runs [VS Code in the browser](https://github.com/linuxserver/docker-code-server) as a sidecar container
alongside Home Assistant. Both containers share the `/config` volume, so you can edit
`configuration.yaml`, automations, and custom components from any browser without needing SSH or
`kubectl exec`.

Enable with:

```yaml
addons:
  codeserver:
    enabled: true
```

### Authentik outpost (`addons.codeserver.authentikOutpost`)

code-server has no meaningful built-in authentication — disabling the password (the default in this
chart) leaves it open to anyone who can reach the URL. An **Authentik outpost** solves this.

An outpost is a lightweight reverse proxy sidecar deployed next to your app. All traffic destined for
code-server hits the outpost first. The outpost calls back to your central
[Authentik](https://goauthentik.io) server to check whether the user has an active, authorised
session. If yes, the request is forwarded to code-server. If not, the user is redirected to the
Authentik login page.

This means code-server gets SSO for free — any identity provider you have wired into Authentik
(LDAP, OIDC, SAML, passkeys) works automatically, with full MFA support.

**Prerequisites:**
- Authentik installed in the cluster
- A *Proxy Provider* configured in Authentik pointing at code-server
- An *Outpost* created in Authentik and the outpost token stored in a secret

**Token secret** — choose one approach:

```yaml
# Option A: existing Kubernetes Secret (key must be named `token`)
addons:
  codeserver:
    authentikOutpost:
      enabled: true
      existingSecret: "my-outpost-token-secret"

# Option B: pull token from an external store via ESO
addons:
  codeserver:
    authentikOutpost:
      enabled: true
      externalSecret:
        enabled: true
        secretStoreRef: onepassword-connect
        remoteKey: authentik-outpost-codeserver   # must have a `token` property
```

### External Secrets (`externalSecrets`)

[External Secrets Operator](https://external-secrets.io) (ESO) syncs secrets from an external store
(1Password, HashiCorp Vault, AWS SSM, Azure Key Vault, …) into Kubernetes `Secret` resources that are
then mounted into the containers.

This chart uses ESO to inject two kinds of secrets into Home Assistant and code-server:

| Secret | Purpose |
|--------|---------|
| `secrets.yaml` | Home Assistant [secrets file](https://www.home-assistant.io/docs/configuration/secrets/) — keeps passwords and tokens out of `configuration.yaml` |
| SSH private key | Lets code-server authenticate to GitHub/GitLab for git push/pull of your HA config |

Items with `coderOnly: true` are mounted only in the code-server container, not in Home Assistant.

```yaml
externalSecrets:
  enabled: true
  items:
  - name: homeassistant-secrets
    secretStoreRef: my-cluster-secret-store
    remoteKey: homeassistant-secrets
    secretKey: secrets.yaml
    defaultMode: 0444           # HA needs read access; LSIO init can't chown read-only mounts
  - name: ha-ssh-key
    secretStoreRef: my-cluster-secret-store
    remoteKey: github-ssh-key
    remoteProperty: private_key
    secretKey: id_ed25519
    coderOnly: true
    mountPath: /mnt/ssh-key/id_ed25519
    defaultMode: 0600
```

### SSH known hosts (`sshKnownHosts`)

When code-server runs a `git push` or `git pull`, SSH must verify the remote server's host key.
In an interactive terminal this shows a prompt — *"Are you sure you want to continue connecting?"*.
Inside a browser-based editor there is no way to answer that prompt, so the git command hangs or
fails.

Pre-loading `known_hosts` removes the prompt by telling SSH which host keys to trust upfront.

```yaml
sshKnownHosts:
  enabled: true
  # Get the value with: ssh-keyscan github.com
  content: "github.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOMqqnkVzrm0SdG6UOoqKLsabgH5C9okWi0dh2l9GKJl"
```

Run `ssh-keyscan <hostname>` for each git host you use (GitHub, GitLab, Gitea, Forgejo, etc.) and
concatenate the lines.

### Gateway API HTTPRoutes (`httproutes`)

Creates `HTTPRoute` resources for use with a [Gateway API](https://gateway-api.sigs.k8s.io) compatible
controller (Cilium, Envoy Gateway, nginx-gateway, Istio, …). Requires the Gateway API CRDs to be
installed in the cluster.

```yaml
httproutes:
- enabled: true
  name: ha-httproute
  hostname: ha.example.com
  path: /
  gateway:
    name: my-gateway
    namespace: kube-system
    sectionName: https-listener
  backend:
    name: home-assistant
    port: 8123
```

### Cilium NetworkPolicy (`networkPolicy`)

Optional `CiliumNetworkPolicy` that locks down pod traffic to only what Home Assistant actually needs.
Requires [Cilium CNI](https://cilium.io).

```yaml
networkPolicy:
  enabled: true
  prometheusNamespace: monitoring   # allow Prometheus scraping
  mqttNamespace: emqx               # allow MQTT broker egress
  postgresNamespace: cnpg-system    # allow PostgreSQL egress (HA recorder)
```

## Prerequisites summary

| Feature | Cluster requirement |
|---------|-------------------|
| `externalSecrets.enabled` | [External Secrets Operator](https://external-secrets.io) |
| `httproutes` | [Gateway API CRDs](https://gateway-api.sigs.k8s.io) |
| `networkPolicy.enabled` | [Cilium CNI](https://cilium.io) |
| `addons.codeserver.authentikOutpost.enabled` | [Authentik](https://goauthentik.io) |
| `hostNetwork: true` | Namespace PSS `privileged` (set automatically by this chart) |

## Values

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| addons | object | `{"codeserver":{"authentikOutpost":{"authentikHost":"","authentikHostBrowser":"","enabled":false,"existingSecret":"","externalSecret":{"enabled":false,"remoteKey":"","secretStoreRef":""},"image":{"repository":"ghcr.io/goauthentik/proxy","tag":"2026.5.3"},"resources":{"limits":{"cpu":"200m","memory":"256Mi"},"requests":{"cpu":"20m","memory":"64Mi"}}},"enabled":false,"env":[{"name":"TZ","value":"UTC"},{"name":"PUID","value":"1000"},{"name":"PGID","value":"1000"},{"name":"DEFAULT_WORKSPACE","value":"/config"},{"name":"PASSWORD","value":""}],"image":{"repository":"lscr.io/linuxserver/code-server","tag":""},"resources":{"limits":{"cpu":"500m","memory":"512Mi"},"requests":{"cpu":"50m","memory":"128Mi"}},"service":{"port":12321,"type":"ClusterIP"}}}` | Optional addons that run as sidecar containers alongside Home Assistant. |
| addons.codeserver | object | `{"authentikOutpost":{"authentikHost":"","authentikHostBrowser":"","enabled":false,"existingSecret":"","externalSecret":{"enabled":false,"remoteKey":"","secretStoreRef":""},"image":{"repository":"ghcr.io/goauthentik/proxy","tag":"2026.5.3"},"resources":{"limits":{"cpu":"200m","memory":"256Mi"},"requests":{"cpu":"20m","memory":"64Mi"}}},"enabled":false,"env":[{"name":"TZ","value":"UTC"},{"name":"PUID","value":"1000"},{"name":"PGID","value":"1000"},{"name":"DEFAULT_WORKSPACE","value":"/config"},{"name":"PASSWORD","value":""}],"image":{"repository":"lscr.io/linuxserver/code-server","tag":""},"resources":{"limits":{"cpu":"500m","memory":"512Mi"},"requests":{"cpu":"50m","memory":"128Mi"}},"service":{"port":12321,"type":"ClusterIP"}}` | VS Code in the browser (code-server), sharing the `/config` volume with Home Assistant. Gives you a full editor for `configuration.yaml`, automations, and custom components without needing shell access to the node. |
| addons.codeserver.authentikOutpost | object | `{"authentikHost":"","authentikHostBrowser":"","enabled":false,"existingSecret":"","externalSecret":{"enabled":false,"remoteKey":"","secretStoreRef":""},"image":{"repository":"ghcr.io/goauthentik/proxy","tag":"2026.5.3"},"resources":{"limits":{"cpu":"200m","memory":"256Mi"},"requests":{"cpu":"20m","memory":"64Mi"}}}` | Authentik proxy outpost for code-server. An outpost is a lightweight reverse proxy deployed next to your app that enforces Authentik authentication before forwarding traffic. It replaces the need for a built-in login screen — any user who hits code-server must first authenticate via your Authentik SSO. The outpost calls back to the Authentik server to validate sessions. Requires [Authentik](https://goauthentik.io) installed in the cluster and a Proxy Provider + Outpost configured for code-server. |
| addons.codeserver.authentikOutpost.authentikHost | string | `""` | Cluster-internal URL of the Authentik server (used by the outpost to validate sessions). |
| addons.codeserver.authentikOutpost.authentikHostBrowser | string | `""` | Public-facing Authentik URL (used for browser redirects to the login page). |
| addons.codeserver.authentikOutpost.enabled | bool | `false` | Enable the Authentik proxy outpost Deployment and Service. |
| addons.codeserver.authentikOutpost.existingSecret | string | `""` | Name of an existing Kubernetes Secret containing the outpost token under key `token`. Mutually exclusive with `externalSecret`. Create this secret manually or via another tool. |
| addons.codeserver.authentikOutpost.externalSecret | object | `{"enabled":false,"remoteKey":"","secretStoreRef":""}` | Fetch the outpost token from an external secret store via ESO instead of an existing secret. |
| addons.codeserver.authentikOutpost.externalSecret.enabled | bool | `false` | Enable ExternalSecret rendering for the outpost token. |
| addons.codeserver.authentikOutpost.externalSecret.remoteKey | string | `""` | Key in the secret store that holds the outpost token (must have a `token` property). |
| addons.codeserver.authentikOutpost.externalSecret.secretStoreRef | string | `""` | ClusterSecretStore name. |
| addons.codeserver.authentikOutpost.image | object | `{"repository":"ghcr.io/goauthentik/proxy","tag":"2026.5.3"}` | Authentik proxy image. |
| addons.codeserver.authentikOutpost.resources | object | `{"limits":{"cpu":"200m","memory":"256Mi"},"requests":{"cpu":"20m","memory":"64Mi"}}` | Resource requests and limits for the outpost proxy container. |
| addons.codeserver.enabled | bool | `false` | Enable the code-server sidecar. |
| addons.codeserver.env | list | `[{"name":"TZ","value":"UTC"},{"name":"PUID","value":"1000"},{"name":"PGID","value":"1000"},{"name":"DEFAULT_WORKSPACE","value":"/config"},{"name":"PASSWORD","value":""}]` | Environment variables for code-server. Runs as the same PUID/PGID as Home Assistant so it has full read/write access to `/config`. |
| addons.codeserver.image | object | `{"repository":"lscr.io/linuxserver/code-server","tag":""}` | code-server container image (LSIO build). |
| addons.codeserver.image.tag | string | `""` | Image tag. Defaults to `Chart.appVersion` when empty. |
| addons.codeserver.resources | object | `{"limits":{"cpu":"500m","memory":"512Mi"},"requests":{"cpu":"50m","memory":"128Mi"}}` | Resource requests and limits for the code-server container. |
| addons.codeserver.service | object | `{"port":12321,"type":"ClusterIP"}` | Service for code-server. Use `ClusterIP` and protect with an Authentik outpost. |
| dnsPolicy | string | `"ClusterFirstWithHostNet"` | DNS policy for the pod. Should be `ClusterFirstWithHostNet` when `hostNetwork` is enabled. |
| env | list | `[{"name":"TZ","value":"UTC"},{"name":"PUID","value":"1000"},{"name":"PGID","value":"1000"}]` | Environment variables injected into the Home Assistant container. The LSIO image requires `PUID`/`PGID` — the HA process runs as UID 1000, not root. |
| externalSecrets | object | `{"enabled":false,"items":[]}` | External Secrets Operator integration for pulling secrets from an external secret store. Requires [ESO](https://external-secrets.io) installed in the cluster. Each item creates an `ExternalSecret` resource that syncs a secret from the store into a Kubernetes `Secret`, which is then mounted into the Home Assistant (and optionally code-server) containers. |
| externalSecrets.enabled | bool | `false` | Enable ExternalSecret rendering. |
| externalSecrets.items | list | `[]` | List of secrets to sync. |
| fullnameOverride | string | `""` | Override the full resource name prefix. |
| hostNetwork | bool | `false` | Enable host network access. Required for mDNS device discovery (Hue, Chromecast, SSDP, Zigbee, etc.). When enabled, the namespace PSS label is automatically set to `privileged`. |
| httproutes | list | `[]` | Gateway API `HTTPRoute` resources. Requires [Gateway API CRDs](https://gateway-api.sigs.k8s.io). |
| image | object | `{"pullPolicy":"IfNotPresent","repository":"lscr.io/linuxserver/homeassistant","tag":""}` | Home Assistant container image. |
| image.pullPolicy | string | `"IfNotPresent"` | Image pull policy. |
| image.repository | string | `"lscr.io/linuxserver/homeassistant"` | Image repository. |
| image.tag | string | `""` | Image tag. Defaults to `Chart.appVersion` when empty. |
| nameOverride | string | `""` | Override the chart name used in resource names and labels. |
| namespaceLabels | object | `{}` | Extra labels merged onto the Namespace created by this chart. Useful for gateway selectors or additional PSS annotations. The Pod Security Standards labels are managed automatically — no need to add them here. |
| networkPolicy | object | `{"enabled":false,"mqttNamespace":"","postgresNamespace":"","prometheusNamespace":""}` | CiliumNetworkPolicy configuration. Requires [Cilium CNI](https://cilium.io). |
| networkPolicy.enabled | bool | `false` | Enable CiliumNetworkPolicy rendering. |
| networkPolicy.mqttNamespace | string | `""` | Namespace of the MQTT broker (e.g. EMQX). When set, adds an egress rule on port 1883. |
| networkPolicy.postgresNamespace | string | `""` | Namespace of the PostgreSQL cluster. When set, adds an egress rule on port 5432. |
| networkPolicy.prometheusNamespace | string | `""` | Namespace where Prometheus runs. When set, adds an ingress rule allowing Prometheus scraping on port 8123. |
| persistence | object | `{"enabled":true,"size":"10Gi","storageClassName":""}` | Persistent storage for the Home Assistant `/config` directory. |
| persistence.enabled | bool | `true` | Enable the PersistentVolumeClaim. |
| persistence.size | string | `"10Gi"` | Storage size. |
| persistence.storageClassName | string | `""` | StorageClass name. Defaults to the cluster default when empty. |
| resources | object | `{"limits":{"cpu":"2","memory":"2Gi"},"requests":{"cpu":"200m","memory":"512Mi"}}` | Resource requests and limits for the Home Assistant container. |
| service | object | `{"annotations":{},"port":8123,"type":"ClusterIP"}` | Kubernetes Service configuration. |
| service.annotations | object | `{}` | Annotations merged onto the Service resource. |
| service.port | int | `8123` | Port to expose. |
| service.type | string | `"ClusterIP"` | Service type. Use `LoadBalancer` to expose HA directly on the LAN. |
| sshKnownHosts | object | `{"content":"","enabled":false}` | SSH `known_hosts` entries pre-loaded into code-server for authenticated git operations. Without this, the first `git push/pull` to a new host blocks on an interactive host-key prompt, which is not possible inside a browser-based editor. Add one line per host. |
| sshKnownHosts.content | string | `""` | Raw known_hosts content (one entry per line). Get entries with: `ssh-keyscan github.com` |
| sshKnownHosts.enabled | bool | `false` | Enable the known_hosts ConfigMap and mount it into code-server. |

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| sironite |  | <https://github.com/sironite> |

## Source Code

* <https://github.com/home-assistant/core>
* <https://github.com/sironite/helm-home-assistant>
