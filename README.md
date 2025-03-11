# Ansible RKE2 Cluster Deployment

This Ansible project automates the deployment of a highly available RKE2 Kubernetes cluster with various integrated components and applications.

## Features

- **High Availability**: Multi-server RKE2 cluster setup
- **Network**: 
  - CNI support (Canal, Cilium, Calico)
  - Load balancing (MetalLB, kube-vip)
  - Ingress (NGINX, Traefik, Kong Ingress Controller)
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
│   ├── my_setup/              # Your cluster configuration
│   │   ├── hosts.yml         # Node inventory and SSH configuration
│   │   └── group_vars/       # Variable files
│   │       └── all.yml       # All configuration variables
│   └── sample/               # Example configurations
├── roles/
│   ├── server/               # RKE2 server setup
│   ├── agent/                # RKE2 agent setup
│   ├── post_server/          # Post-installation components
│   └── apps/                 # Application deployment
├── site.yml                  # Main playbook for infrastructure
├── add_workers.yml           # Playbook for adding worker nodes
└── reset.yml                 # Cluster reset playbook
```

## Configuration

### Variable Structure

All configuration variables are in a single file:

**all.yml**:
- RKE2 version and configuration
- CNI settings
- Load balancer configuration
- Ingress controller setup
- Certificate management
- Storage configuration
- Kubernetes configurations
- Helm settings
- Application-specific variables
- Service configurations

### Deployment Process

The deployment is split into two main phases:

1. **Infrastructure Setup** (`site.yml`):
   ```yaml
   - name: Deploy RKE2 Infrastructure
     hosts: all
     roles:
       - server    # RKE2 server setup
       - agent     # RKE2 agent setup
       - post_server # Post-installation components
   ```

2. **Application Deployment** (`site.yml`):
   ```yaml
   - name: Deploy Applications
     hosts: localhost
     roles:
       - apps      # Application deployment
   ```

## Usage

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd ansible-rke2
   ```

2. Configure your environment:
   - Copy `inventory/sample` to `inventory/my_setup`
   - Update `hosts.yml` with your node information
   - Modify `group_vars/all.yml` with your configurations

3. Deploy the infrastructure:
   ```bash
   ansible-playbook -i inventory/my_setup/hosts.yml site.yml --tags infra
   ```

4. Deploy applications:
   ```bash
   ansible-playbook -i inventory/my_setup/hosts.yml site.yml --tags apps
   ```

5. (Optional) Add worker nodes:
   ```bash
   ansible-playbook -i inventory/my_setup/hosts.yml add_workers.yml
   ```

## Certificate Management

The project supports two types of TLS certificate issuance:

1. **HTTP01 Challenge** (Default)
   - Requires proper ingress configuration
   - Port 80 must be accessible
   - Annotations required:
     ```yaml
     annotations:
       cert-manager.io/cluster-issuer: "letsencrypt-production"
       nginx.ingress.kubernetes.io/ssl-redirect: "false"
       nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
     ```

2. **DNS01 Challenge**
   - Suitable for wildcard certificates
   - No port forwarding required
   - Requires DNS provider API credentials

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
- [x] Structured variable management
- [ ] Air-gapped installation support
- [ ] Backup and disaster recovery
- [ ] Cluster upgrade automation

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 