# Template Development

## Template Format

Each template is a directory containing:

```
appname/
├── app.yaml                    # Metadata & configuration schema (required)
├── docker-compose.yml.tmpl     # Docker Compose template (required)
└── config/                     # Additional config file templates (optional)
```

## Create a New Template

```bash
aradeploy templates new <app>
```

## Validate Templates

```bash
aradeploy templates lint             # Lint all
aradeploy templates lint <app>       # Lint one
```

## app.yaml

Defines the app's metadata, configurable values, ports, volumes, routing, backup behavior, health checks, and system requirements.

```yaml
name: myapp
description: "A brief description of the app"
category: productivity
version: "1.0.0"

values:
  - name: web_port
    description: "Web UI port"
    default: "8080"
  - name: db_password
    description: "Database password"
    secret: true
    auto_gen: password

ports:
  - host: 8080
    container: 80
    protocol: tcp
    description: "Web UI"
    value_name: web_port

volumes:
  - name: data
    container: /app/data
    description: "Application data"

routing:
  container_port: 80
  auth: none                     # none | optional | required

health_check:
  url: "http://localhost:{{.web_port}}"

backup:
  paths:
    - data

requirements:
  min_ram: 256M
  min_disk: 5G
  arch: [amd64, arm64]

dependencies:
  - docker

post_deploy_info:
  access_url: "http://{{.hostname}}:{{.web_port}}"
```

## docker-compose.yml.tmpl

A standard Docker Compose file using Go template syntax for variable substitution.

**User-defined values** from `app.yaml` are accessed via `{{.value_name}}` or `{{index . "value-name"}}` (for hyphenated names).

### System Variables

Automatically available in all templates:

| Variable | Description |
|----------|-------------|
| `{{.hostname}}` | System hostname |
| `{{.domain}}` | Network domain |
| `{{.app_name}}` | App name |
| `{{.data_dir}}` | App data directory path |
| `{{.network}}` | Docker network name |
| `{{.routing_domain}}` | Full routing domain |
| `{{.routing_url}}` | Full routing URL |
| `{{.https_enabled}}` | `"true"` or `"false"` |
| `{{.ca_cert_path}}` | Custom CA certificate path |

### Template Functions

| Function | Description |
|----------|-------------|
| `{{genPassword}}` | Generate random password |
| `{{genUUID}}` | Generate UUID |
| `{{upper .value}}` | Uppercase |
| `{{lower .value}}` | Lowercase |
| `{{replace .value "x" "y"}}` | String replace |
| `{{argon2Hash .password}}` | Argon2id hash |
| `{{default "fallback" .value}}` | Default if empty |

## Required Conventions

All services in `docker-compose.yml.tmpl` should follow these patterns:

```yaml
services:
  myapp:
    image: myapp:{{index . "version"}}
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - aradeploy
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    deploy:
      resources:
        limits:
          memory: 512M
          pids: 100
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:80"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 20s
    labels:
      - "arabackup.enable=true"
      - "arabackup.borg.paths=data"

networks:
  aradeploy:
    external: true
    name: {{index . "network"}}
```

### Database Backup Labels

For database services, add dump labels:

```yaml
labels:
  - "arabackup.enable=true"
  - "arabackup.dump.driver=postgres"    # postgres | mysql | mongodb | sqlite | custom
  - "arabackup.dump.user=postgres"
  - "arabackup.dump.password-env=POSTGRES_PASSWORD"
  - "arabackup.dump.database=mydb"
```
