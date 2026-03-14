# arastack Templates

A collection of Docker Compose application templates for [arastack](https://github.com/jdillenberger/arastack) — a self-hosted homelab management suite.

Each template defines everything needed to deploy, route, monitor, and back up a self-hosted application with a single command:

```bash
aradeploy deploy nextcloud
```

## Available Templates

| Category | Templates |
|----------|-----------|
| **Infrastructure** | adguard, authelia, authentik, crowdsec, keycloak, minio, nginx-proxy-manager, portainer, traefik |
| **Monitoring** | beszel, grafana, loki, uptime-kuma |
| **Media** | audiobookshelf, immich, jellyfin, kavita, komga, navidrome, photoprism |
| **Productivity** | bookstack, calibre-web, docmost, excalidraw, hedgedoc, obsidian, outline, paperless-ngx, stirling-pdf, trilium, wikijs |
| **Development** | code-server, forgejo, gitea, gitlab |
| **Communication** | gotify, mastodon, matrix, mattermost, ntfy |
| **Finance & PM** | actual-budget, firefly-iii, leantime, vikunja |
| **Web & CMS** | ghost, wordpress |
| **Home & Lifestyle** | home-assistant, mealie, tandoor-recipes |
| **Networking** | netbird, wireguard |
| **Analytics** | plausible, umami |
| **Other** | changedetection, linkwarden, openclaw, searxng, syncthing, vaultwarden, webtop |

## Usage

### Deploy a template

```bash
aradeploy deploy <app>
```

### List available templates

```bash
aradeploy templates list
```

### Inspect a template

```bash
aradeploy templates info <app>
```

### Customize a template locally

```bash
aradeploy templates export <app>     # Copy to ~/.aradeploy/templates/ for editing
aradeploy templates delete <app>     # Remove local override
```

## Repository Management

Templates are loaded from git repositories. This repo is the default, but additional repos can be added:

```bash
aradeploy repos add <url> [--name <name>] [--ref <branch>]
aradeploy repos list
aradeploy repos update [name]
aradeploy repos remove <name>
```

**Template priority**: local overrides (`~/.aradeploy/templates/`) > repo templates.

## Template Development

See [development.md](development.md) for the template format reference, conventions, and how to create new templates.
