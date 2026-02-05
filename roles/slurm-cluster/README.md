# Slurm Cluster Role

Ansible role for deploying a two-node Slurm HPC (High Performance Computing) cluster on Ubuntu 24.04 LTS.

## Overview

This role installs and configures a minimal Slurm cluster consisting of:
- **1 Controller Node (slurmctld)**: Manages the cluster, schedules jobs, and monitors resources
- **1 Compute Node (slurmd)**: Executes jobs assigned by the controller

## Requirements

- Ubuntu 24.04 LTS (Noble Numbat) or Ubuntu 22.04 LTS (Jammy Jellyfish)
- Ansible 2.9+
- `ansible.posix` collection (for NFS mount module)
- `community.general` collection (for UFW firewall module, optional)
- Root/sudo access on target hosts
- Network connectivity between nodes
- Shared network storage (see below)

## Shared Network Storage Requirements

Slurm clusters require shared storage accessible by all nodes for proper operation. This is **critical** for:

1. **Job Input/Output Data**: Users need a common location to store job scripts, input files, and retrieve output
2. **User Home Directories**: Should be accessible from both controller and compute nodes
3. **Application Binaries**: Shared software installations ensure consistency across nodes

### Recommended NFS Configuration

You need an NFS server that exports a shared filesystem to all cluster nodes:

| Export Path | Purpose | Minimum Size | Recommended Size |
|-------------|---------|--------------|------------------|
| `/export/slurm` | Job data, scratch space | 100 GB | 500 GB+ |
| `/export/home` | User home directories | 50 GB | 200 GB+ |
| `/export/apps` | Shared applications | 50 GB | 100 GB+ |

### NFS Server Setup (Example)

On your NFS server, configure `/etc/exports`:

```
/export/slurm    10.0.0.0/24(rw,sync,no_subtree_check,no_root_squash)
/export/home     10.0.0.0/24(rw,sync,no_subtree_check)
/export/apps     10.0.0.0/24(ro,sync,no_subtree_check)
```

Then export and start the NFS service:

```bash
exportfs -ra
systemctl enable --now nfs-kernel-server
```

### Alternative Storage Options

- **GlusterFS**: Distributed storage for larger clusters
- **Lustre**: High-performance parallel filesystem for HPC workloads
- **BeeGFS**: Parallel filesystem optimized for HPC
- **Ceph**: Distributed storage with multiple access methods

### Important Notes on Shared Storage

1. **Slurm State Directory**: The `StateSaveLocation` (`/var/spool/slurmctld`) should be on local storage for the controller, or on highly reliable shared storage if using HA
2. **Slurm Spool Directory**: The `SlurmdSpoolDir` (`/var/spool/slurm/slurmd`) must be on **local storage** for each compute node
3. **Configuration Files**: `/etc/slurm/slurm.conf` must be identical on all nodes (this role handles this)

## Node Instance Sizing

### Controller Node (slurmctld)

The controller node manages scheduling and cluster state. Resource requirements depend on cluster size.

| Cluster Size | vCPUs | RAM | Storage | Instance Type (AWS) | Instance Type (Azure) |
|--------------|-------|-----|---------|--------------------|-----------------------|
| 2-10 nodes | 2 | 4 GB | 50 GB SSD | t3.medium | Standard_B2s |
| **Recommended for this role** | **2** | **4 GB** | **50 GB SSD** | **t3.medium** | **Standard_B2s** |

**Controller Node Specifications:**
- **Minimum**: 2 vCPUs, 4 GB RAM, 50 GB storage
- **Recommended**: 2 vCPUs, 4 GB RAM, 50 GB SSD
- **Network**: 1 Gbps minimum

### Compute Node (slurmd)

The compute node runs user jobs. Size according to your workload requirements.

| Workload Type | vCPUs | RAM | Storage | Instance Type (AWS) | Instance Type (Azure) |
|---------------|-------|-----|---------|--------------------|-----------------------|
| Light (testing/dev) | 2-4 | 8 GB | 50 GB | t3.large | Standard_B2ms |
| **General Purpose** | **4** | **16 GB** | **100 GB SSD** | **m5.xlarge** | **Standard_D4s_v3** |
| Memory Intensive | 4-8 | 32-64 GB | 100 GB | r5.xlarge | Standard_E4s_v3 |
| CPU Intensive | 8-16 | 32 GB | 100 GB | c5.2xlarge | Standard_F8s_v2 |
| GPU Workloads | 4-8 | 32 GB | 200 GB | p3.2xlarge | Standard_NC6s_v3 |

**Compute Node Specifications (General Purpose):**
- **Minimum**: 2 vCPUs, 8 GB RAM, 50 GB storage
- **Recommended**: 4 vCPUs, 16 GB RAM, 100 GB SSD
- **Network**: 1 Gbps minimum (10 Gbps for data-intensive workloads)

### On-Premises Hardware Recommendations

| Node Type | CPU | RAM | Storage | Network |
|-----------|-----|-----|---------|---------|
| Controller | Intel Xeon / AMD EPYC (4 cores) | 8 GB ECC | 100 GB SSD | 1 GbE |
| Compute | Intel Xeon / AMD EPYC (8+ cores) | 32 GB ECC | 256 GB NVMe | 10 GbE |

## Role Variables

### Required Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `slurm_node_role` | Node role: `controller` or `compute` | `compute` |
| `slurm_controller_hostname` | Hostname of the controller node | `slurm-controller` |
| `slurm_compute_nodes` | List of compute nodes with specs | See defaults |

### Network Storage Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `slurm_nfs_server` | NFS server IP/hostname | `""` |
| `slurm_nfs_export` | NFS export path | `/export/slurm` |
| `slurm_shared_storage_path` | Local mount point | `/shared` |
| `slurm_nfs_mount_options` | NFS mount options | `rw,sync,hard,intr,nfsvers=4` |
| `slurm_nfs_timeout` | NFS server availability check timeout (seconds) | `30` |

### Firewall Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `slurm_configure_firewall` | Enable automatic UFW firewall configuration | `false` |

When `slurm_configure_firewall` is set to `true`, the role will automatically:
- Install UFW if not present
- Open required ports for Slurm communication (6817/tcp, 6818/tcp)

### Cluster Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `slurm_cluster_name` | Name of the Slurm cluster | `mycluster` |
| `slurm_partition_name` | Default partition name | `batch` |
| `slurm_munge_key` | Base64-encoded Munge key | `""` (auto-generated) |
| `slurm_cred_type` | Credential type for job credentials | `cred/none` |

### Timer Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `slurm_slurmctld_timeout` | Controller heartbeat timeout (seconds) | `120` |
| `slurm_slurmd_timeout` | Compute node heartbeat timeout (seconds) | `300` |
| `slurm_return_to_service` | Automatic node return to service | `1` |
| `slurm_inactive_limit` | Inactive job timeout (0=unlimited) | `0` |
| `slurm_kill_wait` | Time to wait for job termination (seconds) | `30` |
| `slurm_min_job_age` | Minimum time to keep completed jobs (seconds) | `300` |

### Full Variable Reference

See [defaults/main.yml](defaults/main.yml) for all available variables.

## Example Inventory

```ini
[slurm_controller]
controller01 ansible_host=10.0.0.10 slurm_node_role=controller

[slurm_compute]
compute01 ansible_host=10.0.0.11 slurm_node_role=compute

[slurm_cluster:children]
slurm_controller
slurm_compute

[slurm_cluster:vars]
slurm_controller_hostname=controller01
slurm_controller_ip=10.0.0.10
slurm_nfs_server=10.0.0.5
```

## Example Playbook

### Basic Two-Node Cluster

```yaml
---
- name: Deploy Slurm Controller
  hosts: slurm_controller
  become: true
  roles:
    - role: slurm-cluster
      vars:
        slurm_node_role: controller
        slurm_cluster_name: "mycluster"
        slurm_controller_hostname: "controller01"
        slurm_controller_ip: "10.0.0.10"
        slurm_nfs_server: "10.0.0.5"
        slurm_nfs_export: "/export/slurm"
        slurm_compute_nodes:
          - name: "compute01"
            cpus: 4
            memory: 16384

- name: Deploy Slurm Compute Node
  hosts: slurm_compute
  become: true
  roles:
    - role: slurm-cluster
      vars:
        slurm_node_role: compute
        slurm_cluster_name: "mycluster"
        slurm_controller_hostname: "controller01"
        slurm_controller_ip: "10.0.0.10"
        slurm_nfs_server: "10.0.0.5"
        slurm_nfs_export: "/export/slurm"
        slurm_compute_nodes:
          - name: "compute01"
            cpus: 4
            memory: 16384
```

### Using Group Variables

Create `group_vars/slurm_cluster.yml`:

```yaml
---
slurm_cluster_name: "production"
slurm_controller_hostname: "controller01"
slurm_controller_ip: "10.0.0.10"

slurm_nfs_server: "10.0.0.5"
slurm_nfs_export: "/export/slurm"
slurm_shared_storage_path: "/shared"

slurm_compute_nodes:
  - name: "compute01"
    cpus: 4
    memory: 16384
    state: "UNKNOWN"
```

Then your playbook simplifies to:

```yaml
---
- name: Deploy Slurm Cluster
  hosts: slurm_cluster
  become: true
  roles:
    - slurm-cluster
```

## Automatic Health Checks

The role automatically performs the following health checks during deployment:

### Controller Node
- Verifies Munge service is running
- Tests Munge authentication (`munge -n`)
- Waits for slurmctld port availability
- Runs `sinfo` to verify cluster status

### Compute Nodes
- Verifies Munge service is running
- Tests Munge authentication
- Verifies slurmd service is running
- Runs `slurmd -C` to display detected resources
- Tests controller connectivity (`scontrol ping`)

## Post-Installation Verification

After running the playbook, verify the cluster:

```bash
# On controller node
sinfo                    # Show partition and node status
scontrol show nodes      # Detailed node information
scontrol show partition  # Partition configuration

# Test job submission
srun -N1 hostname        # Run hostname on one node
sbatch --wrap="sleep 10" # Submit a batch job
squeue                   # View job queue
```

## Troubleshooting

### Common Issues

1. **Nodes showing as DOWN**
   ```bash
   scontrol update nodename=compute01 state=resume
   ```

2. **Munge authentication failures**
   - Ensure `/etc/munge/munge.key` is identical on all nodes
   - Check permissions: `ls -la /etc/munge/munge.key` (should be 0400, owned by munge)
   - Test Munge: `munge -n | unmunge` (should succeed on all nodes)

3. **slurmd fails to start**
   - Check logs: `journalctl -u slurmd`
   - Verify slurm.conf syntax: `slurmd -C`
   - Ensure controller is reachable: `scontrol ping`

4. **NFS mount issues**
   - Verify NFS server is reachable: `showmount -e <nfs_server>`
   - Check firewall rules for NFS ports (2049/tcp)
   - The role pre-checks NFS availability before mounting

5. **Playbook execution order**
   - **Important**: Controller nodes must be configured before compute nodes
   - The Munge key is generated on the controller and distributed to compute nodes
   - Run the playbook with controller in the first play, then compute nodes

### Log Locations

| Service | Log File |
|---------|----------|
| slurmctld | `/var/log/slurm/slurmctld.log` |
| slurmd | `/var/log/slurm/slurmd.log` |
| munge | `/var/log/munge/munged.log` |

## Security Considerations

1. **Munge Key**: The Munge key is sensitive. Store it securely (e.g., Ansible Vault)
2. **Firewall**: Open only required ports between cluster nodes:
   - 6817/tcp (slurmctld)
   - 6818/tcp (slurmd)
   - 2049/tcp (NFS)
3. **User Access**: Limit SSH access to authorized users only

## License

MIT

## Author

Heath Provost
