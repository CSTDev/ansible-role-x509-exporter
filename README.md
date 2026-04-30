# ansible-role-x509-exporter

Ansible role to install and configure the [enix/x509-exporter](https://github.com/enix/x509-exporter) — a Prometheus exporter that monitors TLS certificate expiry from files on disk.

![Molecule Tests](https://github.com/cstdev/ansible-role-x509-exporter/actions/workflows/molecule.yml/badge.svg)

## Requirements

- Ansible >= 2.14
- A systemd-based Linux host
- Internet access to download the binary from GitHub releases (or pre-stage it)

## Role Variables

All variables are defined in [defaults/main.yml](defaults/main.yml) and can be overridden.

| Variable | Default | Description |
|---|---|---|
| `x509_exporter_version` | `3.16.0` | Version of x509-exporter to install |
| `x509_exporter_arch` | *(auto-detected)* | Target architecture; auto-mapped from `ansible_architecture` (`amd64`, `arm64`) |
| `x509_exporter_os` | `linux` | Target OS |
| `x509_exporter_download_url` | *(derived)* | Full URL to the release archive; override to use a mirror or pre-staged URL |
| `x509_exporter_checksum` | *(derived)* | Checksum for the archive; resolves to the upstream `.sha256` URL so it tracks arch and version automatically |
| `x509_exporter_user` | `x509-exporter` | System user the service runs as |
| `x509_exporter_group` | `x509-exporter` | System group for the service user |
| `x509_exporter_install_dir` | `/usr/local/bin` | Directory to install the binary into |
| `x509_exporter_port` | `9793` | Port to expose metrics on |
| `x509_exporter_listen_address` | `0.0.0.0` | Address to listen on |
| `x509_exporter_watched_paths` | `[/etc/ssl/certs]` | List of directories to scan for certificates |
| `x509_exporter_watched_files` | `[]` | List of individual certificate files to watch (`--watch-file`) |
| `x509_exporter_extra_flags` | `[]` | Additional CLI flags to pass to the exporter |

## Dependencies

None.

## Example Playbook

Minimal usage:

```yaml
- hosts: servers
  become: true
  roles:
    - role: cstdev.x509_exporter
```

With custom configuration:

```yaml
- hosts: servers
  become: true
  roles:
    - role: cstdev.x509_exporter
      vars:
        x509_exporter_version: "3.16.0"
        x509_exporter_port: 9793
        x509_exporter_watched_paths:
          - /etc/ssl/certs
          - /etc/nginx/ssl
        x509_exporter_watched_files:
          - /etc/letsencrypt/live/example.com/fullchain.pem
```

## Testing

Tests use [Molecule](https://molecule.readthedocs.io/) with the Docker driver.

```bash
pip install -r requirements.txt
molecule test
```

The default scenario spins up an Ubuntu 22.04 container, converges the role, then verifies:
- The binary is installed and executable
- The systemd service unit is present, enabled, and active
- The system user and group exist
- The `/metrics` endpoint returns x509 certificate metrics

## License

MIT

## Author

[CSTDev](https://github.com/cstdev)
