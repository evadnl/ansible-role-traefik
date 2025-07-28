Traefik
=======

Ansible role to install and configure Traefik reverse proxy on Debian-based systems.

Requirements
------------

- Ansible 2.9 or higher
- Debian-based Linux distribution (Ubuntu, Debian)
- systemd service manager
- Internet connection to download Traefik binary from GitHub releases

Role Variables
--------------

Available variables with their default values (see `defaults/main.yml`):

```yaml
traefik_version: "3.1.7"                    # Traefik version to install
traefik_architecture: "auto-detected"       # CPU architecture (auto-detected from ansible_architecture)
traefik_user: "traefik"                     # System user for Traefik
traefik_group: "traefik"                    # System group for Traefik
traefik_home_dir: "/opt/traefik"             # Traefik home directory
traefik_config_dir: "/etc/traefik"          # Configuration directory
traefik_data_dir: "/var/lib/traefik"        # Data directory
traefik_log_dir: "/var/log/traefik"         # Log directory
traefik_binary_path: "/usr/local/bin/traefik" # Binary installation path
traefik_service_enabled: true               # Enable systemd service
traefik_service_state: "started"            # Service state (started/stopped)
```

The role automatically detects CPU architecture:
- `x86_64` → `amd64`
- `aarch64` → `arm64`

Dependencies
------------

None.

Example Playbook
----------------

Basic usage:

```yaml
- hosts: servers
  become: yes
  roles:
    - traefik
```

With custom variables:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: traefik
      traefik_version: "3.0.0"
      traefik_service_enabled: true
```

License
-------

MIT-0
