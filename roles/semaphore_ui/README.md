# Semaphore UI Role

Ansible role for deploying [Semaphore UI](https://semaphoreui.com/) using Podman containers.

## Table of Contents

- [Description](#description)
- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Task Tags](#task-tags)
- [Handlers](#handlers)
- [Examples](#examples)
- [Idempotency](#idempotency)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

## Description

This role deploys Semaphore UI, a modern web-based UI for Ansible, using Podman containers. It provides:

- Containerized Semaphore deployment with BoltDB backend
- Automatic admin user creation
- Optional project, inventory, and repository initialization
- Support for additional user creation
- Full idempotency - safe to run multiple times
- Cleanup mode for complete removal

## Requirements

- Ansible 2.9 or higher
- Podman installed on target host
- `containers.podman` collection

Install the required collection:

```bash
ansible-galaxy collection install containers.podman
```

## Role Variables

### Required Variables

| Variable | Description |
|----------|-------------|
| `semaphore_admin_password` | Admin password (REQUIRED - must be provided) |

### Operation Mode

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_cleanup` | `false` | Set to `true` to remove container and data |

### Container Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_container_name` | `semaphore` | Container name |
| `semaphore_image` | `docker.io/semaphoreui/semaphore:latest` | Container image |
| `semaphore_port` | `3000` | Port inside container |
| `semaphore_host_port` | `3000` | Port exposed on host |
| `semaphore_restart_policy` | `always` | Container restart policy |

### Volume Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_volume_name` | `semaphore_data` | Persistent data volume name |
| `semaphore_git_volume` | `""` | Path to mount at /git (defaults to playbook_dir) |

### Admin User Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_admin_user` | `admin` | Admin username |
| `semaphore_admin_email` | `admin@localhost` | Admin email |
| `semaphore_admin_name` | `Admin User` | Admin display name |

### Security Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_access_key_encryption` | `""` | Encryption key (auto-generated if empty) |

### Project Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_create_project` | `true` | Whether to create initial project |
| `semaphore_project_name` | `Ansible Tests` | Project name |

### Inventory Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_create_inventory` | `true` | Whether to create initial inventory |
| `semaphore_inventory_name` | `localhost` | Inventory name |
| `semaphore_inventory_content` | localhost static | Inventory content |

### Repository Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_create_repository` | `true` | Whether to create initial repository |
| `semaphore_repository_name` | `Local Playbooks` | Repository name |
| `semaphore_repository_path` | `/git` | Repository path in container |

### Additional Users

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_additional_users` | `[]` | List of additional users to create |

Each user in the list requires:
- `name`: Display name
- `username`: Login username
- `email`: Email address
- `password`: User password
- `admin`: (optional) Whether user is admin (default: false)

### API Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `semaphore_api_host` | `localhost` | API host for connections |
| `semaphore_api_url` | auto-constructed | Full API URL |
| `semaphore_wait_timeout` | `120` | Max wait time for API (seconds) |
| `semaphore_wait_delay` | `5` | Delay between API checks (seconds) |
| `semaphore_api_retries` | `3` | Retries for API operations |
| `semaphore_api_retry_delay` | `2` | Delay between retries (seconds) |

## Task Tags

| Tag | Description |
|-----|-------------|
| `semaphore` | All role tasks |
| `semaphore_validate` | Input validation only |
| `semaphore_setup` | Container and API setup |
| `semaphore_cleanup` | Remove container and data |

## Handlers

The role provides handlers for container lifecycle management:

| Handler | Description |
|---------|-------------|
| `restart semaphore` | Restart the container and wait for API |
| `stop semaphore` | Stop the container |
| `recreate semaphore` | Recreate container with current configuration |

### Using Handlers

```yaml
- name: Update configuration and restart
  ansible.builtin.copy:
    content: "{{ config_content }}"
    dest: /some/config/path
  notify: restart semaphore
```

## Examples

### Basic Setup

```yaml
- name: Deploy Semaphore UI
  hosts: localhost
  roles:
    - role: semaphore_ui
      vars:
        semaphore_admin_password: "MySecurePassword123"
```

### Setup with Additional Users

```yaml
- name: Deploy Semaphore UI with users
  hosts: localhost
  roles:
    - role: semaphore_ui
      vars:
        semaphore_admin_password: "{{ vault_semaphore_admin_pass }}"
        semaphore_project_name: "Production Playbooks"
        semaphore_additional_users:
          - name: "John Doe"
            username: "jdoe"
            email: "jdoe@example.com"
            password: "{{ vault_jdoe_pass }}"
            admin: false
          - name: "Jane Smith"
            username: "jsmith"
            email: "jsmith@example.com"
            password: "{{ vault_jsmith_pass }}"
            admin: true
```

### Container-Only Setup (No Project)

```yaml
- name: Deploy Semaphore container only
  hosts: localhost
  roles:
    - role: semaphore_ui
      vars:
        semaphore_admin_password: "MySecurePassword123"
        semaphore_create_project: false
        semaphore_create_inventory: false
        semaphore_create_repository: false
```

### Custom Port and Remote Access

```yaml
- name: Deploy Semaphore on custom port
  hosts: semaphore_server
  roles:
    - role: semaphore_ui
      vars:
        semaphore_admin_password: "MySecurePassword123"
        semaphore_host_port: 8080
        semaphore_api_host: "{{ ansible_default_ipv4.address }}"
```

### Cleanup

```yaml
- name: Remove Semaphore UI
  hosts: localhost
  roles:
    - role: semaphore_ui
      vars:
        semaphore_cleanup: true
```

### Using Tags

```bash
# Run only validation
ansible-playbook site.yml --tags semaphore_validate

# Run setup without validation
ansible-playbook site.yml --tags semaphore_setup

# Run only cleanup
ansible-playbook site.yml --tags semaphore_cleanup -e semaphore_cleanup=true
```

## Idempotency

This role is designed to be idempotent:

- **Container**: Podman modules handle container state properly
- **Projects**: Checks for existing project before creating
- **Inventories**: Checks for existing inventory before creating
- **Repositories**: Checks for existing repository before creating
- **Users**: Checks for existing username before creating

Running the role multiple times with the same configuration will not create duplicate resources.

## Testing

This role includes Molecule tests for verification.

### Prerequisites

```bash
pip install molecule molecule-podman ansible-core
```

### Running Tests

```bash
# Run default test scenario (setup and verify)
molecule test

# Run cleanup scenario
molecule test -s cleanup

# Run converge only (no destroy)
molecule converge

# Run verification only
molecule verify

# Login to test instance for debugging
molecule login
```

### Test Scenarios

| Scenario | Description |
|----------|-------------|
| `default` | Full setup with project, inventory, repository, and users |
| `cleanup` | Tests setup followed by cleanup to verify removal |

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "semaphore_admin_password is required" | Password not provided | Pass via `-e semaphore_admin_password=...` |
| "Podman is not installed" | Podman not in PATH | Install Podman on target host |
| API timeout | Container slow to start | Increase `semaphore_wait_timeout` |
| Port already in use | Another service on port | Change `semaphore_host_port` |

### Checking Container Status

```bash
# View container status
podman ps -a | grep semaphore

# View container logs
podman logs semaphore

# Check API health
curl http://localhost:3000/api/ping
```

### Accessing the UI

After successful deployment, access Semaphore UI at:
```
http://<host>:<semaphore_host_port>
```

Default: `http://localhost:3000`

## License

MIT

## Author

Heath Provost
