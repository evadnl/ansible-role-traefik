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
# Installation variables
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

# Traefik configuration variables
traefik_global_send_anonymous_usage: false  # Send anonymous usage statistics
traefik_global_check_new_version: true      # Check for new Traefik versions

traefik_log_level: "WARN"                   # Log level (DEBUG, INFO, WARN, ERROR, FATAL, PANIC)

traefik_api_dashboard: true                  # Enable web dashboard
traefik_api_insecure: true                   # Allow insecure dashboard access

# EntryPoints configuration
traefik_entrypoint_web_enabled: true        # Enable HTTP entrypoint
traefik_entrypoint_web_port: 80             # HTTP port
traefik_entrypoint_websecure_enabled: true  # Enable HTTPS entrypoint
traefik_entrypoint_websecure_port: 443      # HTTPS port

# Provider configuration
traefik_provider_docker_enabled: true       # Enable Docker provider
traefik_provider_docker_exposed_by_default: false # Expose containers by default
traefik_provider_docker_watch: true         # Watch for Docker changes
traefik_provider_docker_endpoint: "unix:///var/run/docker.sock" # Docker socket

traefik_provider_file_enabled: true         # Enable file provider
traefik_provider_file_directory: "/etc/traefik/conf/" # File provider directory
traefik_provider_file_watch: true           # Watch file provider directory

# Certificate resolvers
traefik_certificate_resolvers_enabled: true # Enable certificate resolvers
traefik_certificate_resolvers:              # Certificate resolver configuration
  cloudflare:
    email: "email@example.com"              # ACME email address
    storage: "/certs/acme.json"             # Certificate storage path
    ca_server: "https://acme-v02.api.letsencrypt.org/directory" # ACME CA server
    dns_challenge:
      provider: "cloudflare"                # DNS challenge provider
      resolvers:
        - "1.1.1.1:53"                     # DNS resolvers
        - "8.8.8.8:53"
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
