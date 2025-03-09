# RKE2 Cluster Deployment with Ansible

This Ansible project deploys a high-availability RKE2 Kubernetes cluster with support for both internet-connected and air-gapped environments, kube-vip for HA control plane, and MetalLB for load balancing.

## Features

- Support for both internet-connected and air-gapped deployments
- Private registry integration
- High-availability control plane with kube-vip
- MetalLB for bare metal load balancing
- Automated deployment with Ansible
- Comprehensive system prerequisite checks
- Separate worker node addition process

## Prerequisites

### System Requirements
- Ansible 2.9+
- Target nodes with:
  - Minimum 4GB RAM
  - Minimum 2 vCPUs
  - Linux OS (RHEL 8+, Ubuntu 20.04+)
  - SSH access
  - Sudo privileges

### Network Requirements
- All nodes must be able to communicate with each other
- Required ports:
  - 6443: Kubernetes API Server
  - 9345: RKE2 server join
  - 8472: VXLAN (Canal/Flannel)
  - 10250: Kubelet metrics
  - 2379-2380: etcd client and peer ports

### Air-gapped Requirements (if applicable)
- Access to a private registry
- RKE2 installation artifacts:
  - RKE2 binary
  - RKE2 images tarball

## Configuration

1. Update `inventory/hosts.yml` with your server and worker node details:
   ```yaml
   rke2_servers:
     hosts:
       rke2-server-1:
       rke2-server-2:
       rke2-server-3:
   rke2_agents:
     hosts:
       rke2-agent-1:
       # Add more worker nodes as needed
   ```

2. Modify `group_vars/all.yml` to set:
   - Deployment mode (air-gapped or internet-connected)
   - Air gap configuration (if applicable)
   - Private registry details (if applicable)
   - RKE2 version (currently set to v1.31.5+rke2r1)
   - kube-vip virtual IP
   - MetalLB address pool
   - Network configuration
   - Proxy settings (if needed)

## Deployment Modes

### Internet-Connected Deployment
1. Set `airgap_mode: false` in `group_vars/all.yml`
2. Deploy the control plane:
   ```bash
   # Basic deployment
   ansible-playbook site.yml

   # Deploy and copy kubeconfig with VIP
   ansible-playbook site.yml --tags kubeconfig

   # Deploy and run validation
   ansible-playbook site.yml --tags validate

   # Deploy with both kubeconfig and validation
   ansible-playbook site.yml --tags "kubeconfig,validate"
   ```
3. Add worker nodes (optional):
   ```bash
   ansible-playbook add_workers.yml
   ```

### Air-Gapped Deployment
1. Download RKE2 artifacts on a node with internet access:
   ```bash
   # Set the RKE2 version
   export RKE2_VERSION="v1.31.5+rke2r1"
   
   # Download artifacts
   curl -OL https://github.com/rancher/rke2/releases/download/${RKE2_VERSION}/rke2-images.linux-amd64.tar.gz
   curl -OL https://github.com/rancher/rke2/releases/download/${RKE2_VERSION}/rke2.linux-amd64.tar.gz
   ```

2. Transfer artifacts to your air-gapped environment

3. Set `airgap_mode: true` in `group_vars/all.yml` and configure:
   - `rke2_artifact_path`
   - `private_registry_url`
   - Other air-gap related settings

4. Deploy the cluster:
   ```bash
   # Basic deployment
   ansible-playbook site.yml

   # Deploy and copy kubeconfig with VIP
   ansible-playbook site.yml --tags kubeconfig

   # Deploy with validation
   ansible-playbook site.yml --tags validate
   ```

## Optional Features

### Kubeconfig Management
To get the kubeconfig file configured with the kube-vip address:
```bash
# During deployment
ansible-playbook site.yml --tags kubeconfig

# After deployment
ansible-playbook site.yml --tags kubeconfig --skip-tags never
```

The kubeconfig will be:
- Copied to `./kubeconfig`
- Configured to use the kube-vip address
- Set with proper permissions (0600)
- Ready to use with kubectl

### Cluster Validation
To validate the cluster status:
```bash
# During deployment
ansible-playbook site.yml --tags validate

# After deployment
ansible-playbook site.yml --tags validate --skip-tags never
```

The validation will check:
- Node status
- Pod status across all namespaces
- Component health
- Add-on deployment status

### Worker Node Management
The `add_workers.yml` playbook allows you to add worker nodes to an existing cluster:

1. Add worker nodes to inventory under `rke2_agents` group
2. Run the worker addition playbook:
   ```bash
   ansible-playbook add_workers.yml
   ```

Worker node deployment tags:
```bash
# Run only prerequisite checks
ansible-playbook add_workers.yml --tags prereq

# Run only agent installation
ansible-playbook add_workers.yml --tags agent
```

## LXC VM Management

### Prerequisites
- LXD installed on the host machine
- Proper network configuration
- SSH key pair for VM access

### Configuration
Update `group_vars/all.yml` VM configuration:
```yaml
vm_config:
  vm_prefix: "rke2-server"
  vm_count: 3
  vm_cpu: 2
  vm_memory: "4GiB"
  vm_disk: "8GiB"
  username: "rke2"
  password: "rke2@123"
  timezone: "Asia/Dubai"
  network_interface: "enp5s0"
  storage_pool: "default"
```

### Creating VMs
To create the RKE2 server and agent VMs:
```bash
# Create all VMs (3 servers + 1 agent)
ansible-playbook -i inventory/vm_host.yml lxc.yml --tags create

# Verify VM creation
lxc list
```

### Deleting VMs
To delete the RKE2 server and agent VMs:
```bash
# Delete all VMs
ansible-playbook -i inventory/vm_host.yml lxc.yml --tags delete
```

## Verification

1. Check node connectivity:
   ```bash
   ansible all -m ping
   ```

2. Run prerequisite checks:
   ```bash
   ansible-playbook site.yml --tags prereq
   ```

3. Verify cluster status:
   ```bash
   export KUBECONFIG=${PWD}/kubeconfig
   kubectl get nodes
   ```

4. Check high-availability components:
   ```bash
   # Verify kube-vip
   kubectl get pods -n kube-system | grep kube-vip

   # Verify MetalLB
   kubectl get pods -n metallb-system
   ```

## Directory Structure

```
.
├── inventory/
│   ├── hosts.yml          # Inventory file for all nodes
│   └── vm_host.yml        # VM host inventory
├── group_vars/
│   └── all.yml           # Global variables
├── roles/
│   ├── common/           # System prerequisites and checks
│   ├── lxc/             # LXC/LXD VM management
│   └── rke2/
│       ├── server/      # Control plane configuration
│       └── agent/       # Worker node configuration
├── add_workers.yml       # Worker node addition playbook
├── site.yml             # Main cluster deployment playbook
├── lxc.yml              # LXC VM management playbook
└── README.md
```

## Notes

- The kube-vip virtual IP (VIP) must be accessible from all nodes
- MetalLB address pool should not conflict with existing network services
- For production deployments, ensure proper security measures are in place
- In air-gapped mode, ensure all required images are available in your private registry
- System prerequisites are automatically checked before deployment
- Worker nodes can be added at any time after the control plane is ready
- The cluster automatically verifies node readiness during deployment
- Optional features can be enabled using appropriate tags
- Use `--skip-tags never` when running tagged tasks after initial deployment
- LXC VMs can be managed using the `lxc.yml` playbook with appropriate tags
- VM configuration can be customized in `group_vars/all.yml` 