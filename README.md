# Ansible RKE2 Cluster Deployment

This Ansible project automates the deployment of a highly available RKE2 Kubernetes cluster with various integrated components and applications.

## Features

- **High Availability**: Multi-server RKE2 cluster setup
- **Network**: 
  - CNI support (Canal, Cilium, Calico)
  - Load balancing (MetalLB, kube-vip)
  - Ingress (NGINX, Traefik)
- **Storage**: NFS CSI driver integration
- **Security**: 
  - TLS certificate management with cert-manager
  - Let's Encrypt integration with HTTP01 and DNS01 challenge support
  - Wildcard certificate support
- **Monitoring**: 
  - Prometheus stack
  - Grafana
  - Loki logging
- **GPU Support**: NVIDIA GPU operator integration
- **Applications**:
  - Langfuse deployment support
  - MinIO object storage (optional)

## Prerequisites

- Ansible 2.9+
- SSH access to target nodes
- Sudo privileges on target nodes
- DNS records configured for your domain
- (Optional) DNS provider API credentials for wildcard certificates

## Project Structure

```
ansible-rke2/
├── inventory/
│   └── my_setup/
│       ├── hosts.yml           # Node inventory and SSH configuration
│       └── group_vars/
│           └── all.yml         # Main configuration variables
├── roles/
│   ├── server/                 # RKE2 server setup
│   │   ├── tasks/
│   │   │   ├── main.yml       # Main server tasks
│   │   │   ├── cni.yml        # CNI configuration
│   │   │   ├── kubevip.yml    # kube-vip setup
│   │   │   └── metallb.yml    # MetalLB setup
│   │   └── templates/
│   │       └── config.yaml.j2 # RKE2 server config
│   │       └── kubevip.yaml.j2    # kubevip setup
│   │       └── metallb.yaml.j2    # MetalLB setup
│   │       └── canal-config.yaml.j2    # Canal config
│   ├── agent/                  # RKE2 agent setup
│   │   ├── tasks/
│   │   │   └── main.yml       # Agent node tasks
│   │   └── templates/
│   │       └── config.yaml.j2 # RKE2 agent config
│   └── post_server/           # Post-installation components
│       ├── tasks/
│       │   ├── main.yml
│       │   └── cert-manager.yml
│       │   └── langfuse.yml
│       │   └── minio.yml
│       │   └── prometheus.yml
│       │   └── grafana.yml
│       │   └── loki.yml
│       │   └── gpu.yml
│       │   └── nfs.yml
│       │   └── csi-nfs.yml
│       │   └── metallb.yml
│       │   └── ingress.yml
│       └── templates/
│           ├── cert-manager.yml.j2
│           ├── cert-manager-issuer.yml.j2
│           ├── langfuse.yml.j2
│           ├── minio.yml.j2
│           ├── kic.yml.j2
│           ├── grafana.yml.j2
│           ├── loki.yml.j2
│           ├── metallb.yml.j2
│           ├── csi-nfs.yml.j2
│           ├── gpu-operator.yml.j2
│           ├── ingress-nginx.yml.j2
│           ├── monitoring.yml.j2
```

## Configuration

### Main Configuration (group_vars/all.yml)

Key configuration sections:
- RKE2 version and basic setup
- Network configuration (CNI, load balancers)
- TLS and certificate management
- Monitoring stack settings
- Application-specific configurations

### Node Configuration (hosts.yml)

- Server and agent node definitions
- Node labels and taints
- SSH access configuration

## Certificate Management

The project supports two types of TLS certificate issuance:

1. **HTTP01 Challenge** (Default)
   - Individual certificates per domain
   - No DNS provider configuration needed
   - Automatic HTTP challenge handling

2. **DNS01 Challenge** (For Wildcards)
   - Wildcard certificate support
   - Supported providers:
     - Cloudflare
     - GoDaddy
     - Namecheap
   - Requires DNS provider API credentials

## Usage

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd ansible-rke2
   ```

2. Configure your inventory:
   - Copy `inventory/my_setup` to `inventory/your-setup`
   - Update `hosts.yml` with your node information
   - Modify `group_vars/all.yml` with your desired configuration

3. Run the playbook:
   ```bash
   ansible-playbook -i inventory/your-setup/hosts.yml site.yml
   ```

## Current Status

- [x] Basic RKE2 cluster deployment
- [x] High availability setup
- [x] CNI integration
- [x] Load balancer setup
- [x] Certificate management
- [x] Monitoring stack
- [x] GPU operator support
- [x] NFS CSI driver
- [x] Langfuse application deployment
- [ ] Air-gapped installation support (in progress)
- [ ] Backup and disaster recovery
- [ ] Cluster upgrade automation

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 