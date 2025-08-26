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
traefik_certs_dir: "/etc/traefik/certs"     # Certificates directory
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
traefik_api_dashboard_hostname: ""           # Custom hostname for dashboard (empty = use raw IP, no TLS)

# EntryPoints configuration
traefik_entrypoint_http_enabled: true       # Enable HTTP entrypoint
traefik_entrypoint_http_port: 80            # HTTP port
traefik_entrypoint_https_enabled: true      # Enable HTTPS entrypoint
traefik_entrypoint_https_port: 443         # HTTPS port

# Custom EntryPoints
traefik_custom_entrypoints: {}              # Define custom entrypoints
# Example:
# traefik_custom_entrypoints:
#   ssh:
#     address: ":2222"
#   mqtt:
#     address: ":1883"
#   custom_web:
#     address: ":8080"
#     transport:
#       respondingTimeouts:
#         readTimeout: "60s"
#     http:
#       healthcheck: "/health"
#   grpc:
#     address: ":9090"
#     http:
#       healthcheck: "/grpc.health.v1.Health/Check"

# Provider configuration
traefik_provider_docker_enabled: true       # Enable Docker provider
traefik_provider_docker_exposed_by_default: false # Expose containers by default
traefik_provider_docker_watch: true         # Watch for Docker changes
traefik_provider_docker_endpoint: "unix:///var/run/docker.sock" # Docker socket

traefik_provider_file_enabled: true         # Enable file provider
traefik_provider_file_directory: "/etc/traefik/conf/" # File provider directory
traefik_provider_file_watch: true           # Watch file provider directory

# Services configuration (creates separate files per service)
traefik_services: {}                        # Define services, routers, and middlewares
# Example:
# traefik_services:
#   api:
#     protocol: "http"                      # Protocol: "http" (default) or "tcp"
#     routers:
#       api:
#         rule: "Host(`api.example.com`)"
#         service: "api"
#         entrypoints: ["https"]
#         tls:
#           certResolver: "cloudflare"
#     services:
#       api:
#         loadBalancer:
#           servers:
#             - url: "http://192.168.1.100:3000"
#             - url: "http://192.168.1.101:3000"
#           healthCheck:
#             path: "/health"
#             interval: "10s"
#   web:
#     protocol: "http"                      # HTTP routing (default if not specified)
#     routers:
#       web:
#         rule: "Host(`example.com`) || Host(`www.example.com`)"
#         service: "web"
#         entrypoints: ["http", "https"]
#         middlewares: ["redirect-to-https"]
#         tls:
#           certResolver: "cloudflare"
#     services:
#       web:
#         loadBalancer:
#           servers:
#             - url: "http://192.168.1.200:80"
#     middlewares:
#       redirect-to-https:
#         redirectScheme:
#           scheme: "https"
#           permanent: true
#   tcp-service:
#     protocol: "tcp"                       # TCP routing
#     routers:
#       tcp-router:
#         rule: "HostSNI(`tcp.example.com`)"
#         service: "tcp-backend"
#         entrypoints: ["tcp-port"]
#         tls: {}                           # Enable TLS passthrough
#     services:
#       tcp-backend:
#         loadBalancer:
#           servers:
#             - address: "192.168.1.100:3306"  # Use address for TCP services
#             - address: "192.168.1.101:3306"

# Certificate resolvers
traefik_certificate_resolvers_enabled: true # Enable certificate resolvers
traefik_certificate_resolvers:              # Certificate resolver configuration
  cloudflare:
    email: "email@example.com"              # ACME email address
    CLOUDFLARE_DNS_API_TOKEN: ""            # Cloudflare DNS API token (see Security section)
    storage: "{{ traefik_certs_dir }}/acme.json" # Certificate storage path
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

Security
--------

**⚠️ Important: Secret Management**

The `CLOUDFLARE_DNS_API_TOKEN` variable contains sensitive credentials that should never be exposed in plain text or committed to version control. Always use secure secret management practices:

- **Ansible Vault**: Encrypt the token using `ansible-vault` by creating a vault.yml file and adding the `vault_CLOUDFLARE_DNS_API_TOKEN` in there.

- **External Secret Managers**: Use HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, etc.

- **Environment Variables**: Set the token via environment variables on the target system

- **Git Exclusion**: Ensure tokens are never committed to git repositories

Example using Ansible Vault:
```yaml
traefik_certificate_resolvers:
  cloudflare:
    email: "email@example.com"
    CLOUDFLARE_DNS_API_TOKEN: {{ vault_CLOUDFLARE_DNS_API_TOKEN }}
```

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
