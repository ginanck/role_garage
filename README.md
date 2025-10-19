# Ansible Role: Garage

[![Ansible Role](https://img.shields.io/badge/ansible--galaxy-garage-blue.svg)](https://galaxy.ansible.com/your-namespace/garage)
[![License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)

An Ansible role for installing and configuring [Garage](https://garagehq.deuxfleurs.fr/), a lightweight, S3-compatible object storage server designed for self-hosting and distributed deployments.

## Features

- **Binary Installation**: Downloads and installs Garage binaries directly from official releases
- **Multi-Architecture Support**: Supports x86_64, aarch64, armv7l, and i386 architectures
- **Single-Node Ready**: Automated single-node cluster setup and initialization
- **Extensible Design**: Structured for easy extension to multi-node cluster configurations
- **Security Hardened**: Systemd service with security restrictions and proper user isolation
- **Configurable**: Comprehensive variable system for customizing all aspects of Garage
- **Version Independent**: Easy version management and upgrades

## Requirements

- **OS**: Linux distributions with systemd support
- **Ansible**: 2.10 or higher
- **Architecture**: x86_64, aarch64, armv7l, or i386
- **Network**: Internet access for binary downloads
- **Privileges**: Root/sudo access for system configuration

## Supported Platforms

- Ubuntu 20.04+ (focal, jammy, noble)
- Debian 11+ (bullseye, bookworm)
- EL 8/9 (RHEL, CentOS Stream, Rocky Linux, AlmaLinux)
- Fedora 37+

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install your-namespace.garage
```

### From Git Repository

```bash
git clone https://github.com/your-username/ansible-role-garage.git roles/garage
```

## Quick Start

### Basic Single-Node Setup

```yaml
---
- hosts: garage-server
  become: true
  roles:
    - role: garage
      vars:
        garage_version: "v2.1.0"
        garage_s3_api:
          s3_region: "my-region"
          root_domain: ".s3.myserver.com"
```

This will:
1. Install Garage v2.1.0 binary
2. Create system user and directories
3. Generate secure configuration
4. Start and enable the service
5. Initialize single-node cluster
6. Configure S3 API endpoint

## Role Variables

### Core Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `garage_version` | `"v2.1.0"` | Garage version to install |
| `garage_bin_path` | `"/usr/local/bin/garage"` | Binary installation path |
| `garage_config_dir` | `"/etc/garage"` | Configuration directory |
| `garage_user` | `"garage"` | System user for service |
| `garage_group` | `"garage"` | System group for service |

### Storage Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `garage_metadata_dir` | `"/var/lib/garage/meta"` | Metadata storage path |
| `garage_data_dir` | `"/var/lib/garage/data"` | Object data storage path |
| `garage_db_engine` | `"sqlite"` | Database engine (sqlite, lmdb, sled) |

### Network Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `garage_rpc_bind_addr` | `"[::]:3901"` | RPC bind address |
| `garage_rpc_public_addr` | `"{{ ansible_default_ipv4.address }}:3901"` | RPC public address |
| `garage_s3_api.api_bind_addr` | `"[::]:3900"` | S3 API bind address |
| `garage_s3_web.bind_addr` | `"[::]:3902"` | Web interface bind address |
| `garage_admin_api.api_bind_addr` | `"[::]:3903"` | Admin API bind address |

### Cluster Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `garage_replication_factor` | `1` | Data replication factor |
| `garage_cluster_mode` | `"single"` | Cluster mode (single, cluster) |
| `garage_bootstrap_cluster` | `true` | Auto-initialize cluster |
| `garage_layout_zone` | `"dc1"` | Node zone assignment |
| `garage_layout_capacity` | `"1G"` | Node storage capacity |

### Feature Flags

| Variable | Default | Description |
|----------|---------|-------------|
| `garage_features.metrics_enabled` | `true` | Enable metrics endpoint |
| `garage_features.admin_api_enabled` | `true` | Enable admin API |
| `garage_features.k2v_enabled` | `true` | Enable K2V API |
| `garage_features.website_enabled` | `true` | Enable static website hosting |

## Usage Examples

### Single-Node Production Setup

```yaml
---
- hosts: garage-server
  become: true
  roles:
    - role: garage
      vars:
        garage_version: "v2.1.0"
        garage_metadata_dir: "/opt/garage/meta"
        garage_data_dir: "/opt/garage/data"
        garage_db_engine: "lmdb"
        garage_s3_api:
          s3_region: "production"
          api_bind_addr: "0.0.0.0:3900"
          root_domain: ".s3.example.com"
        garage_s3_web:
          bind_addr: "0.0.0.0:3902"
          root_domain: ".web.example.com"
        garage_admin_api:
          api_bind_addr: "127.0.0.1:3903"  # Admin API only on localhost
        garage_log_level: "info"
```

### Custom Configuration Override

```yaml
- hosts: garage-server
  become: true
  roles:
    - role: garage
      vars:
        garage_custom_config:
          block_size: "1048576"
          sled_cache_capacity: "134217728"
          sled_flush_every_ms: "2000"
```

### Development Setup with Debug Logging

```yaml
- hosts: garage-dev
  become: true
  roles:
    - role: garage
      vars:
        garage_version: "v2.1.0"
        garage_log_level: "debug"
        garage_db_engine: "sqlite"
        garage_features:
          metrics_enabled: true
          admin_api_enabled: true
          k2v_enabled: true
```

## Multi-Node Cluster Setup (Future Extension)

The role is designed to support multi-node clusters. Here's the planned configuration:

```yaml
# This is a preview of future multi-node support
- hosts: garage-cluster
  become: true
  roles:
    - role: garage
      vars:
        garage_cluster_mode: "cluster"
        garage_replication_factor: 3
        garage_bootstrap_cluster: false  # Manual cluster init required
        garage_cluster_nodes:
          - name: "node1"
            address: "10.0.1.10:3901"
            zone: "dc1"
            capacity: "100G"
          - name: "node2"
            address: "10.0.1.11:3901"
            zone: "dc1"
            capacity: "100G"
          - name: "node3"
            address: "10.0.1.12:3901"
            zone: "dc2"
            capacity: "100G"
```

## Security Considerations

### Default Security Features

- Dedicated system user with minimal privileges
- Systemd security hardening (NoNewPrivileges, PrivateTmp, etc.)
- Auto-generated secure RPC secrets and admin tokens
- Restrictive file permissions on configuration files
- Network binding controls

### Manual Security Steps

1. **Firewall Configuration**: Configure firewall rules for your ports
2. **TLS Termination**: Use a reverse proxy (nginx, Apache) for HTTPS
3. **Admin API Security**: Restrict admin API access to trusted networks
4. **Backup Security**: Secure your configuration backups (contains secrets)

### Generated Secrets

The role automatically generates:
- RPC secret (64-character hex string)
- Admin API token (32-character alphanumeric)
- Metrics token (32-character alphanumeric)

Store these securely if you need to recreate the installation.

## Maintenance

### Viewing Service Status

```bash
systemctl status garage
journalctl -u garage -f
```

### Cluster Operations

```bash
# Check cluster status
garage -c /etc/garage/garage.toml status

# View layout
garage -c /etc/garage/garage.toml layout show

# Create bucket
garage -c /etc/garage/garage.toml bucket create my-bucket

# Create key
garage -c /etc/garage/garage.toml key create my-key
```

### Upgrading Garage

Simply update the version variable and run the playbook:

```yaml
garage_version: "v2.2.0"  # New version
```

The role will:
1. Download the new binary
2. Stop the service
3. Replace the binary
4. Start the service
5. Verify the installation

## File Structure

```
role_garage/
├── defaults/main.yml          # Default variables
├── handlers/main.yml          # Service handlers
├── meta/main.yml             # Role metadata
├── tasks/
│   ├── main.yml              # Main task orchestration
│   ├── user_setup.yml        # User and directory creation
│   ├── install.yml           # Binary installation
│   ├── configure.yml         # Configuration management
│   ├── service.yml           # Service management
│   └── cluster_init.yml      # Cluster initialization
├── templates/
│   ├── garage.toml.j2        # Main configuration template
│   └── garage.service.j2     # Systemd service template
├── tests/
│   ├── inventory             # Test inventory
│   └── test.yml              # Test playbook
└── README.md                 # This file
```

## Testing

Run the included test playbook:

```bash
ansible-playbook -i tests/inventory tests/test.yml
```

The test validates:
- Binary installation and version
- Service status and health
- Configuration file generation
- Cluster initialization
- API endpoint availability
- Directory permissions

## Troubleshooting

### Common Issues

1. **Service fails to start**
   - Check journalctl logs: `journalctl -u garage -f`
   - Verify configuration: `garage -c /etc/garage/garage.toml --help`
   - Check file permissions on data directories

2. **Cluster initialization fails**
   - Ensure service is running and healthy
   - Check network connectivity on RPC port
   - Verify RPC address configuration

3. **Download fails**
   - Check internet connectivity
   - Verify architecture detection
   - Check GitHub releases availability

### Log Analysis

```bash
# Service logs
journalctl -u garage --since "1 hour ago"

# Configuration validation
garage -c /etc/garage/garage.toml --help

# Cluster status
garage -c /etc/garage/garage.toml status
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Ensure all tests pass
6. Submit a pull request

## License

MIT

## Author Information

This role was created to provide easy deployment of Garage object storage.

## Links

- [Garage Official Website](https://garagehq.deuxfleurs.fr/)
- [Garage Documentation](https://garagehq.deuxfleurs.fr/documentation/)
- [Garage GitHub Repository](https://git.deuxfleurs.fr/Deuxfleurs/garage)
- [Garage Releases](https://garagehq.deuxfleurs.fr/_releases.html)
