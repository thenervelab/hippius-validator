# Hippius Validator Ansible Deployment

This Ansible project provides a professional-grade deployment for multiple blockchain nodes including IPFS, Subtensor, and Hippius validator nodes.

## Structure

```
ansible/
├── inventory/           # Environment-specific inventories
│   ├── production/      # Production environment
│   └── staging/        # Staging environment
├── roles/              # Role-based tasks
│   ├── common/         # System updates and firewall
│   ├── docker/         # Docker installation
│   ├── ipfs/          # IPFS node setup
│   ├── subtensor/     # Subtensor node setup
│   └── hippius/       # Hippius validator setup
├── group_vars/         # Variables for all groups
├── site.yml            # Main playbook
└── README.md           # This file
```

## Requirements

- Ansible 2.9+
- Target hosts running Ubuntu 24.04
- SSH access to target hosts
- Docker and docker-compose installed on control machine

## Components

### IPFS Node
- Runs as dedicated IPFS user
- Configurable API and gateway ports
- Systemd service for automatic startup

### Subtensor Node
- Runs mainnet-lite node via Docker
- Custom port mapping:
  - External RPC: 9945 → Internal: 9944
  - External P2P: 30334 → Internal: 30333
  - External WS: 9934 → Internal: 9933
- Configured with warp sync for faster synchronization

### Hippius Validator
- Runs as root user
- Configured with off-chain worker support
- Default substrate ports (configurable)
- Systemd service for automatic startup
- Automatic ED25519 key management:
  - Uses provided key if specified in `hippius_key`
  - Generates a secure ED25519 key if none provided
  - Stores key in the network directory with proper permissions

## Usage

1. Configure your inventory in `inventory/[environment]/hosts.yml`
2. Review and adjust variables in `group_vars/all.yml`:
   - IPFS configuration
   - Subtensor ports and resources
   - Hippius binary location and ports
3. Run the playbook:

```bash
# For production
ansible-playbook -i inventory/production/hosts.yml site.yml
```

## Configuration

### Important Variables

```yaml
# IPFS Configuration
ipfs_version: "v0.26.0"
ipfs_api_address: "/ip4/127.0.0.1/tcp/5001"
ipfs_gateway_address: "/ip4/127.0.0.1/tcp/8080"

# Subtensor Configuration
subtensor_ports:
  rpc: 9945    # External port for RPC
  p2p: 30334   # External port for P2P
  ws: 9934     # External port for WebSocket

# Hippius Configuration
hippius_binary_url: "https://example.com/hippius-thebrain-latest.tar.gz"
hippius_key: ""  # Optional: Provide ED25519 key in hex format
hippius_ports:
  rpc: 9944
  p2p: 30333
  ws: 9933
```

## Security Notes

- IPFS runs as a dedicated system user
- Subtensor runs in a Docker container with resource limits
- Hippius validator runs as root (as required)
- Firewall rules are automatically configured for all required ports

## Maintenance

### Service Management

```bash
# IPFS
systemctl status ipfs

# Subtensor
docker-compose -f /opt/subtensor/docker-compose.yml ps

# Hippius
systemctl status hippius
```

## Troubleshooting

Check service logs:
```bash
# IPFS logs
journalctl -u ipfs -f

# Subtensor logs
docker-compose -f /opt/subtensor/docker-compose.yml logs -f

# Hippius logs
journalctl -u hippius -f
```
