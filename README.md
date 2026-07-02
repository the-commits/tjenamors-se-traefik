[![sourcehut](https://img.shields.io/badge/sourcehut-~the--commits/tjenamors--se--traefik-2d6b9e?logo=sourcehut)](https://git.sr.ht/~the-commits/tjenamors-se-traefik)
[![GitHub mirror](https://img.shields.io/badge/GitHub-the--commits/tjenamors--se--traefik-181717?logo=github)](https://github.com/the-commits/tjenamors-se-traefik)

# Traefik

Ansible role for deploying [Traefik](https://traefik.io) reverse proxy via Docker.

## Requirements

### Server Requirements

#### Minimum
- 64-bit x86 (x86_64/amd64) or ARM64 CPU
- at least 512 MB of RAM
- 1 vCPU
- 10 GB of hard drive space
- Docker Engine and Docker Compose installed

### Ansible Requirements
- Ansible 2.14+
- Root/sudo access on target hosts

## Installation

### From Ansible Galaxy (once published)

```bash
ansible-galaxy install tjenamors.traefik
```

### From source

Add to your `requirements.yml`:

```yaml
---
roles:
  - name: traefik
    src: git@git.sr.ht:~the-commits/tjenamors-se-traefik
    scm: git
    version: main
```

Then install:

```bash
ansible-galaxy install -r requirements.yml -p roles/
```

## Role Variables

All variables are prefixed with `traefik_`. See [`defaults/main.yml`](defaults/main.yml) for the full list.

| Variable | Default | Description |
|---|---|---|
| `traefik_base_dir` | `/opt/traefik` | Installation directory |
| `traefik_image` | `traefik:latest` | Traefik Docker image |
| `traefik_http_port` | `80` | HTTP entrypoint port |
| `traefik_https_port` | `443` | HTTPS entrypoint port |
| `traefik_web_enabled` | `true` | Enable HTTP entrypoint (redirects to HTTPS) |
| `traefik_websecure_enabled` | `true` | Enable HTTPS entrypoint |
| `traefik_dashboard_enabled` | `false` | Enable Traefik dashboard |
| `traefik_acme_email` | `""` | Let's Encrypt email (required for TLS) |
| `traefik_acme_staging` | `false` | Use staging CA for testing |
| `traefik_acme_challenge` | `http` | Challenge type (`http`, `tls`, `dns`) |
| `traefik_domains` | `[]` | List of domains for file-based routing |
| `traefik_docker_network` | `traefik` | Docker network name |
| `traefik_memory_max` | `512m` | Hard memory limit |
| `traefik_memory_reservation` | `256m` | Soft memory reservation |
| `traefik_cpus_max` | `1` | Hard CPU limit |
| `traefik_cpus_reservation` | `0.5` | Soft CPU reservation |
| `traefik_log_level` | `INFO` | Log level |

## Dependencies

Requires Docker Engine and Docker Compose. See
[`tjenamors-se-docker`](https://git.sr.ht/~the-commits/tjenamors-se-docker) and
[`tjenamors-se-docker-compose`](https://git.sr.ht/~the-commits/tjenamors-se-docker-compose).

## Example Playbook

```yaml
- hosts: all
  become: true
  vars:
    traefik_acme_email: admin@example.com
    traefik_domains:
      - example.com
      - app.example.com
    traefik_dashboard_enabled: true
  roles:
    - role: traefik
```

## Integrating with Other Roles

Other roles (e.g. `azuracast`) can depend on Traefik by adding labels to their
Docker Compose services:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.app.rule=Host(`app.example.com`)"
  - "traefik.http.routers.app.tls=true"
  - "traefik.http.routers.app.tls.certResolver=letsencrypt"
  - "traefik.http.routers.app.entrypoints=websecure"
```

Services must join the shared `traefik` network:

```yaml
networks:
  traefik:
    external: true
```

## Service Management (systemd)

```bash
sudo systemctl status traefik
sudo systemctl start traefik
sudo systemctl stop traefik
sudo systemctl restart traefik
sudo journalctl -u traefik
```

## Local Development

### Prerequisites

```bash
pip install molecule molecule-plugins[vagrant]
vagrant plugin install vagrant-libvirt
```

### Running Tests

```bash
molecule test
```

## License

AGPL-3.0 — see [LICENSE](LICENSE) for details.
