# Ansible Role: semaphore-api-get

An Ansible role for making GET requests to Semaphore UI API endpoints. This role supports retrieving data from inventory, task, variable groups, repository, keys, templates, and schedules endpoints with built-in error handling, retry logic, and query parameter support.

## Table of Contents

- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Supported Endpoints](#supported-endpoints)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [Return Values](#return-values)
- [Authentication](#authentication)
- [Advanced Features](#advanced-features)
  - [Error Handling and Retry Logic](#error-handling-and-retry-logic)
  - [Query Parameters](#query-parameters)
  - [Debug Output Control](#debug-output-control)
  - [Response Validation](#response-validation)
- [API Endpoint Details](#api-endpoint-details)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

## Requirements

- Ansible 2.9 or higher
- Access to a running Semaphore UI instance
- Valid authentication credentials (token or username/password)

## Role Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `semaphore_api_endpoint` | The API endpoint to call | `inventory`, `task`, `variable_groups`, `repository`, `keys`, `templates`, `schedules` |
| `semaphore_project_id` | The Semaphore project ID | `1` |
| `semaphore_api_base_url` | Base URL for the Semaphore API | `http://localhost:3000/api` |

### Authentication Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `semaphore_api_token` | API token for authentication | `""` |
| `semaphore_api_user` | Username for basic auth | `admin` |
| `semaphore_api_password` | Password for basic auth | `changeme` |

**Note**: If `semaphore_api_token` is provided, it will be used for authentication. Otherwise, basic authentication with username and password will be used.

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `semaphore_resource_id` | Specific resource ID to retrieve | `""` (empty = get all) |
| `semaphore_api_query_params` | Dictionary of query parameters to add to the API URL | `{}` |
| `semaphore_api_retries` | Number of retry attempts for failed API calls | `3` |
| `semaphore_api_retry_delay` | Delay in seconds between retry attempts | `5` |
| `semaphore_api_debug` | Enable debug output for API calls | `true` |
| `semaphore_api_verbose_debug` | Enable verbose debug output (includes full response) | `false` |
| `semaphore_api_validate_certs` | Validate SSL certificates | `true` |
| `semaphore_api_timeout` | HTTP request timeout in seconds | `30` |
| `semaphore_api_follow_redirects` | Follow HTTP redirects | `all` |
| `semaphore_api_return_content` | Return response content | `true` |
| `semaphore_api_status_code` | Expected HTTP status code | `200` |

### Facts Set by Role

The role automatically sets the `semaphore_ui_api` fact from the `semaphore_api_base_url` if available. This allows integration with other roles that set up Semaphore.

## Supported Endpoints

### 1. Inventory Endpoint
Retrieves inventory configurations for a project.

**Endpoint**: `inventory`
**API Path**: `/api/project/{project_id}/inventory[/{inventory_id}]`
**Returns**: List of inventories or a specific inventory

### 2. Task Endpoint
Retrieves task execution history and status.

**Endpoint**: `task`
**API Path**: `/api/project/{project_id}/tasks[/{task_id}]`
**Returns**: List of tasks or a specific task

### 3. Variable Groups Endpoint
Retrieves environment variables and variable groups.

**Endpoint**: `variable_groups`
**API Path**: `/api/project/{project_id}/environment[/{env_id}]`
**Returns**: List of variable groups or a specific variable group

### 4. Repository Endpoint
Retrieves repository configurations linked to a project.

**Endpoint**: `repository`
**API Path**: `/api/project/{project_id}/repositories[/{repo_id}]`
**Returns**: List of repositories or a specific repository

### 5. Keys Endpoint
Retrieves SSH keys and access keys for a project.

**Endpoint**: `keys`
**API Path**: `/api/project/{project_id}/keys[/{key_id}]`
**Returns**: List of keys or a specific key

### 6. Templates Endpoint
Retrieves task templates for a project.

**Endpoint**: `templates`
**API Path**: `/api/project/{project_id}/templates[/{template_id}]`
**Returns**: List of templates or a specific template

### 7. Schedules Endpoint
Retrieves scheduled tasks for a project.

**Endpoint**: `schedules`
**API Path**: `/api/project/{project_id}/schedules[/{schedule_id}]`
**Returns**: List of schedules or a specific schedule

## Dependencies

None.

## Example Playbooks

### Example 1: Get All Inventories

```yaml
---
- name: Get all inventories from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "inventory"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display retrieved inventories
      ansible.builtin.debug:
        var: semaphore_inventory_data
```

### Example 2: Get Specific Task by ID

```yaml
---
- name: Get specific task from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "{{ semaphore_ui_api }}"
    semaphore_api_user: "admin"
    semaphore_api_password: "your-password"
    semaphore_project_id: 1
    semaphore_api_endpoint: "task"
    semaphore_resource_id: "42"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display task details
      ansible.builtin.debug:
        var: semaphore_task_data
```

### Example 3: Get All Variable Groups

```yaml
---
- name: Get all variable groups from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://semaphore.example.com:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 2
    semaphore_api_endpoint: "variable_groups"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display variable groups
      ansible.builtin.debug:
        var: semaphore_variable_groups_data
```

### Example 4: Get All Repositories

```yaml
---
- name: Get all repositories from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "repository"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display repositories
      ansible.builtin.debug:
        var: semaphore_repository_data
```

### Example 5: Using with semaphore_ui_api Fact

```yaml
---
- name: Setup Semaphore and retrieve data
  hosts: localhost
  tasks:
    - name: Setup Semaphore UI
      ansible.builtin.include_role:
        name: semaphore_ui
      vars:
        semaphore_admin_password: "secure-password"

    - name: Get inventories using semaphore_ui_api fact
      ansible.builtin.include_role:
        name: semaphore-api-get
      vars:
        semaphore_project_id: 1
        semaphore_api_endpoint: "inventory"
        semaphore_api_token: "{{ semaphore_api_token }}"

    - name: Display retrieved inventories
      ansible.builtin.debug:
        var: semaphore_inventory_data
```

### Example 6: Disable SSL Verification (Development Only)

```yaml
---
- name: Get data with SSL verification disabled
  hosts: localhost
  vars:
    semaphore_api_base_url: "https://semaphore.local:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "inventory"
    semaphore_api_validate_certs: false
  roles:
    - semaphore-api-get
```

### Example 7: Get All Templates

```yaml
---
- name: Get all templates from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "templates"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display templates
      ansible.builtin.debug:
        var: semaphore_templates_data
```

### Example 8: Get All Keys

```yaml
---
- name: Get all SSH keys from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "keys"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display keys
      ansible.builtin.debug:
        var: semaphore_keys_data
```

### Example 9: Using Query Parameters

```yaml
---
- name: Get tasks with query parameters
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "task"
    semaphore_api_query_params:
      sort: "start"
      order: "desc"
  roles:
    - semaphore-api-get
```

### Example 10: Disable Debug Output

```yaml
---
- name: Get data without debug output
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "inventory"
    semaphore_api_debug: false
  roles:
    - semaphore-api-get
```

### Example 11: Get All Schedules

```yaml
---
- name: Get all schedules from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "schedules"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display schedules
      ansible.builtin.debug:
        var: semaphore_schedules_data
```

### Example 12: Custom Retry Configuration

```yaml
---
- name: Get data with custom retry settings
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "task"
    semaphore_api_retries: 5
    semaphore_api_retry_delay: 10
  roles:
    - semaphore-api-get
```

## Return Values

Each endpoint sets a specific fact with the retrieved data:

| Endpoint | Fact Name | Description |
|----------|-----------|-------------|
| `inventory` | `semaphore_inventory_data` | Inventory data from API |
| `task` | `semaphore_task_data` | Task data from API |
| `variable_groups` | `semaphore_variable_groups_data` | Variable groups data from API |
| `repository` | `semaphore_repository_data` | Repository data from API |
| `keys` | `semaphore_keys_data` | SSH keys data from API |
| `templates` | `semaphore_templates_data` | Template data from API |
| `schedules` | `semaphore_schedules_data` | Schedule data from API |

All facts contain the parsed JSON response from the Semaphore API.

## Authentication

The role supports two authentication methods:

### 1. Token Authentication (Recommended)

Set the `semaphore_api_token` variable:

```yaml
vars:
  semaphore_api_token: "your-api-token-here"
```

### 2. Basic Authentication

Set username and password:

```yaml
vars:
  semaphore_api_user: "admin"
  semaphore_api_password: "your-password"
```

**Note**: If both are provided, token authentication takes precedence.

## Advanced Features

### Error Handling and Retry Logic

The role includes built-in error handling and automatic retry logic for API calls:

- **Automatic retries**: Failed API calls are automatically retried up to `semaphore_api_retries` times (default: 3)
- **Configurable delay**: Delay between retries can be set with `semaphore_api_retry_delay` (default: 5 seconds)
- **Clear error messages**: When an API call fails after all retries, a detailed error message is displayed with:
  - HTTP status code
  - Error message from the API
  - The URL that was called
  - Any API error details

Example with custom retry settings:

```yaml
vars:
  semaphore_api_retries: 5
  semaphore_api_retry_delay: 10
```

### Query Parameters

Add custom query parameters to API requests using the `semaphore_api_query_params` dictionary:

```yaml
vars:
  semaphore_api_query_params:
    sort: "name"
    order: "asc"
    limit: "50"
```

Common use cases:
- **Sorting**: `{ "sort": "name", "order": "asc" }`
- **Filtering**: `{ "status": "running" }`
- **Pagination**: `{ "limit": "50", "offset": "100" }`

### Debug Output Control

Control the amount of debug information displayed during execution:

#### Standard Debug Output (Default)

```yaml
vars:
  semaphore_api_debug: true  # Shows API call info and response summary
```

Output includes:
- Endpoint being called
- Project and resource IDs
- Authentication method
- Response size and type

#### Verbose Debug Output

```yaml
vars:
  semaphore_api_verbose_debug: true  # Shows full API response
```

Additional output:
- Complete API response JSON
- Full URL with query parameters
- All HTTP headers

#### Disable Debug Output

```yaml
vars:
  semaphore_api_debug: false  # Minimal output
```

### Response Validation

The role automatically validates API responses:

- **Status code check**: Ensures the response status matches the expected code
- **JSON validation**: Verifies the response is valid JSON
- **Error detection**: Checks for API error messages in the response

If validation fails, the role will:
1. Display a detailed error message
2. Fail the playbook execution
3. Provide troubleshooting information

## API Endpoint Details

### Inventory Endpoint

Returns inventory configurations including:
- Inventory ID
- Inventory name
- Inventory type (static, dynamic, etc.)
- Inventory source
- SSH key associations

**Example Response Structure**:
```json
[
  {
    "id": 1,
    "name": "Production Servers",
    "project_id": 1,
    "inventory": "inventory content here",
    "type": "static",
    "ssh_key_id": 1
  }
]
```

### Task Endpoint

Returns task execution data including:
- Task ID
- Task status
- Task output
- Start and end times
- Template information

**Example Response Structure**:
```json
[
  {
    "id": 42,
    "template_id": 1,
    "status": "success",
    "start": "2024-01-15T10:30:00Z",
    "end": "2024-01-15T10:35:00Z"
  }
]
```

### Variable Groups Endpoint

Returns environment variable groups including:
- Variable group ID
- Variable group name
- Environment variables
- Secrets (if accessible)

**Example Response Structure**:
```json
[
  {
    "id": 1,
    "name": "Production Vars",
    "project_id": 1,
    "variables": [
      {
        "name": "ENVIRONMENT",
        "value": "production"
      }
    ]
  }
]
```

### Repository Endpoint

Returns repository configurations including:
- Repository ID
- Repository name
- Git URL
- Branch information
- SSH key associations

**Example Response Structure**:
```json
[
  {
    "id": 1,
    "name": "Main Ansible Repo",
    "project_id": 1,
    "git_url": "https://github.com/user/repo.git",
    "git_branch": "main",
    "ssh_key_id": 1
  }
]
```

### Keys Endpoint

Returns SSH keys and access keys including:
- Key ID
- Key name
- Key type (ssh, login_password, none)
- Public key (for SSH keys)

**Example Response Structure**:
```json
[
  {
    "id": 1,
    "name": "Production SSH Key",
    "project_id": 1,
    "type": "ssh",
    "secret": "encrypted_private_key_data"
  }
]
```

### Templates Endpoint

Returns task templates including:
- Template ID
- Template name
- Playbook filename
- Inventory, repository, and environment associations
- Template type (task, build, deploy)

**Example Response Structure**:
```json
[
  {
    "id": 1,
    "name": "Deploy Application",
    "project_id": 1,
    "playbook": "deploy.yml",
    "inventory_id": 1,
    "repository_id": 1,
    "environment_id": 1,
    "type": "deploy"
  }
]
```

### Schedules Endpoint

Returns scheduled task executions including:
- Schedule ID
- Cron expression
- Template association
- Next run time
- Enabled status

**Example Response Structure**:
```json
[
  {
    "id": 1,
    "project_id": 1,
    "template_id": 1,
    "cron_format": "0 2 * * *",
    "active": true
  }
]
```

## Testing

A basic test playbook is available in the tests directory:

```bash
ansible-playbook tests/test.yml
```

## Troubleshooting

### Common Issues

1. **401 Unauthorized**: Check your API token or credentials
2. **404 Not Found**: Verify the project ID and resource ID exist
3. **SSL Certificate Error**: Set `semaphore_api_validate_certs: false` for self-signed certs (development only)
4. **Connection Timeout**: Increase `semaphore_api_timeout` value

### Debug Mode

Enable debug output by running with verbose flags:

```bash
ansible-playbook your-playbook.yml -vvv
```

## License

MIT

## Author Information

This role was created for managing Semaphore UI API interactions in Ansible automation workflows.

## Contributing

Contributions are welcome! Please submit issues or pull requests for any improvements or bug fixes.
