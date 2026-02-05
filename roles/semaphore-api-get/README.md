# Ansible Role: semaphore-api-get

An Ansible role for making GET requests to Semaphore UI API endpoints. This role supports retrieving data from 13 different endpoints with built-in error handling, retry logic, and query parameter support.

## Table of Contents

- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Supported Endpoints](#supported-endpoints)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
- [Return Values](#return-values)
- [Authentication](#authentication)
- [Vault Token Management](#vault-token-management)
- [Advanced Features](#advanced-features)
  - [Error Handling and Retry Logic](#error-handling-and-retry-logic)
  - [Query Parameters](#query-parameters)
  - [Debug Output Control](#debug-output-control)
  - [Response Validation](#response-validation)
  - [Response Metadata](#response-metadata)
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
| `semaphore_api_endpoint` | The API endpoint to call | See [Supported Endpoints](#supported-endpoints) |
| `semaphore_api_base_url` | Base URL for the Semaphore API | `http://localhost:3000/api` |

### Conditionally Required Variables

| Variable | Description | Required When |
|----------|-------------|---------------|
| `semaphore_project_id` | The Semaphore project ID | Project-scoped endpoints |
| `semaphore_resource_id` | Specific resource ID | `task_output` endpoint |

### Authentication Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `semaphore_api_token` | API token for authentication | `""` |
| `semaphore_api_user` | Username for basic auth | `admin` |
| `semaphore_api_password` | Password for basic auth | `changeme` |

**Note**: If `semaphore_api_token` is provided, it will be used for authentication. Otherwise, basic authentication with username and password will be used.

### Vault Token Management Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `semaphore_api_vault_file` | Path to Ansible vault file containing token | `{{ playbook_dir }}/secrets/vault.yml` |
| `semaphore_api_vault_password_file` | Vault password file path | `""` |
| `semaphore_api_auto_create_token` | Auto-create token if none exists | `true` |

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
| `semaphore_api_expected_status_codes` | List of acceptable HTTP status codes | `[200]` |
| `semaphore_api_validate_json` | Validate response is JSON | `true` |
| `semaphore_api_store_metadata` | Store response metadata in fact | `true` |

### Facts Set by Role

The role automatically sets the `semaphore_ui_api` fact from the `semaphore_api_base_url` if available. This allows integration with other roles that set up Semaphore.

## Supported Endpoints

### Project-Scoped Endpoints (require `semaphore_project_id`)

| Endpoint | Description | API Path |
|----------|-------------|----------|
| `inventory` | Inventory configurations | `/api/project/{id}/inventory` |
| `task` | Task execution history | `/api/project/{id}/tasks` |
| `variable_groups` | Environment variables | `/api/project/{id}/environment` |
| `repository` | Repository configurations | `/api/project/{id}/repositories` |
| `keys` | SSH/access keys | `/api/project/{id}/keys` |
| `templates` | Task templates | `/api/project/{id}/templates` |
| `schedules` | Scheduled tasks | `/api/project/{id}/schedules` |
| `events` | Activity/event log | `/api/project/{id}/events` |
| `views` | Project views | `/api/project/{id}/views` |
| `task_output` | Task output (requires `resource_id`) | `/api/project/{id}/tasks/{task_id}/output` |

### Global Endpoints (do not require `semaphore_project_id`)

| Endpoint | Description | API Path |
|----------|-------------|----------|
| `projects` | List all projects | `/api/projects` |
| `users` | List all users (admin) | `/api/users` |
| `user` | Current user info | `/api/user` |

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

### Example 3: Get Task Output

```yaml
---
- name: Get output from a specific task
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "task_output"
    semaphore_resource_id: "42"  # Required for task_output
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display task output
      ansible.builtin.debug:
        var: semaphore_task_output_data
```

### Example 4: List All Projects (Global Endpoint)

```yaml
---
- name: Get all projects from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_api_endpoint: "projects"
    # Note: project_id not required for global endpoints
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display all projects
      ansible.builtin.debug:
        var: semaphore_projects_data
```

### Example 5: Get Current User Info

```yaml
---
- name: Get current authenticated user info
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_api_endpoint: "user"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display current user
      ansible.builtin.debug:
        var: semaphore_current_user_data
```

### Example 6: Get Project Events/Activity Log

```yaml
---
- name: Get project activity log
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "events"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display project events
      ansible.builtin.debug:
        var: semaphore_events_data
```

### Example 7: Using Query Parameters

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

### Example 8: Get All Users (Admin Only)

```yaml
---
- name: Get all users from Semaphore
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-admin-api-token"
    semaphore_api_endpoint: "users"
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display all users
      ansible.builtin.debug:
        var: semaphore_users_data
```

### Example 9: Get All Templates

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

### Example 10: Custom Retry Configuration

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

### Example 11: Disable SSL Verification (Development Only)

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

### Example 12: Access Response Metadata

```yaml
---
- name: Get data and check response metadata
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_token: "your-api-token-here"
    semaphore_project_id: 1
    semaphore_api_endpoint: "inventory"
    semaphore_api_store_metadata: true
  roles:
    - semaphore-api-get

  post_tasks:
    - name: Display response metadata
      ansible.builtin.debug:
        msg:
          - "Status: {{ semaphore_api_last_response.status }}"
          - "Response time: {{ semaphore_api_last_response.elapsed }} seconds"
          - "URL called: {{ semaphore_api_last_response.url }}"
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
| `events` | `semaphore_events_data` | Event/activity log data from API |
| `views` | `semaphore_views_data` | View data from API |
| `task_output` | `semaphore_task_output_data` | Task output data from API |
| `projects` | `semaphore_projects_data` | Projects data from API |
| `users` | `semaphore_users_data` | Users data from API |
| `user` | `semaphore_current_user_data` | Current user data from API |

All facts contain the parsed JSON response from the Semaphore API.

### Response Metadata

When `semaphore_api_store_metadata: true` (default), the role also sets:

```yaml
semaphore_api_last_response:
  status: 200              # HTTP status code
  elapsed: 0.123           # Response time in seconds
  url: "http://..."        # Full URL called
  endpoint: "inventory"    # Endpoint name
  content_type: "application/json"
```

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

## Vault Token Management

The role supports automatic API token management using Ansible vault files. This enables secure token storage and automatic token creation when needed.

### Vault Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `semaphore_api_vault_file` | Path to Ansible vault file | `{{ playbook_dir }}/secrets/vault.yml` |
| `semaphore_api_vault_password_file` | Vault password file path | `""` |
| `semaphore_api_auto_create_token` | Auto-create token if none exists | `true` |

### How It Works

1. **Check for existing token**: If `semaphore_api_token` is already set, use it directly
2. **Load from vault**: If vault file exists, decrypt and load `semaphore_api_token`
3. **Create new token**: If no token found and `semaphore_api_auto_create_token` is true, authenticate with username/password and create a new token
4. **Save to vault**: Newly created tokens are automatically saved to the vault file
5. **Fatal error**: If no token available and vault doesn't exist, fail with helpful message

### Setup

First, create the vault file:

```bash
mkdir -p secrets
ansible-vault create secrets/vault.yml --vault-password-file .vault_pass
# Add: semaphore_api_token: ""
```

### Example: Using Vault Token

```yaml
---
- name: Get data using vault-stored token
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_vault_file: "{{ playbook_dir }}/secrets/vault.yml"
    semaphore_api_vault_password_file: ".vault_pass"
    semaphore_api_endpoint: "projects"
  roles:
    - semaphore-api-get
```

### Example: Auto-Create Token

```yaml
---
- name: Auto-create and store API token
  hosts: localhost
  vars:
    semaphore_api_base_url: "http://localhost:3000/api"
    semaphore_api_vault_file: "{{ playbook_dir }}/secrets/vault.yml"
    semaphore_api_vault_password_file: ".vault_pass"
    semaphore_api_user: "admin"
    semaphore_api_password: "{{ vault_admin_password }}"
    semaphore_api_auto_create_token: true
    semaphore_api_endpoint: "projects"
  roles:
    - semaphore-api-get
```

### Disabling Vault Integration

To disable vault integration entirely:

```yaml
vars:
  semaphore_api_vault_file: ""
  semaphore_api_token: "your-token-here"
```

## Advanced Features

### Error Handling and Retry Logic

The role includes built-in error handling with human-readable error messages:

- **Automatic retries**: Failed API calls are automatically retried up to `semaphore_api_retries` times (default: 3)
- **Configurable delay**: Delay between retries can be set with `semaphore_api_retry_delay` (default: 5 seconds)
- **Status-specific messages**: Clear error messages for common HTTP status codes (401, 403, 404, 429, 5xx)
- **Troubleshooting hints**: Contextual troubleshooting suggestions based on the error type

Example error output:
```
API call failed for inventory endpoint.

Status Code: 404
Error: Not Found - The requested resource does not exist
URL: http://localhost:3000/api/project/999/inventory

Troubleshooting: Check that project_id=999 and resource_id=N/A exist.
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
- Response size, type, and time

#### Verbose Debug Output

```yaml
vars:
  semaphore_api_verbose_debug: true  # Shows full API response
```

Additional output:
- Complete API response JSON
- Full URL with query parameters

#### Disable Debug Output

```yaml
vars:
  semaphore_api_debug: false  # Minimal output
```

### Response Validation

The role automatically validates API responses:

- **Status code check**: Ensures the response status matches expected codes
- **JSON validation**: Verifies the response is valid JSON (configurable with `semaphore_api_validate_json`)
- **Error detection**: Checks for API error messages in the response

### Response Metadata

Access detailed information about the last API call:

```yaml
- name: Check response time
  ansible.builtin.debug:
    msg: "API responded in {{ semaphore_api_last_response.elapsed }} seconds"
```

## API Endpoint Details

### Project-Scoped Endpoints

#### Inventory Endpoint
Returns inventory configurations including inventory ID, name, type, source, and SSH key associations.

#### Task Endpoint
Returns task execution data including task ID, status, output, timestamps, and template information.

#### Variable Groups Endpoint
Returns environment variable groups including variables and secrets.

#### Repository Endpoint
Returns repository configurations including Git URL, branch, and SSH key associations.

#### Keys Endpoint
Returns SSH keys and access keys including key type and public key data.

#### Templates Endpoint
Returns task templates including playbook filename, associations, and template type.

#### Schedules Endpoint
Returns scheduled task executions including cron expression and enabled status.

#### Events Endpoint
Returns project activity log including user actions and timestamps.

#### Views Endpoint
Returns project views for organizing templates.

#### Task Output Endpoint
Returns the output/logs for a specific task execution. Requires `semaphore_resource_id` to be set.

### Global Endpoints

#### Projects Endpoint
Returns all projects the authenticated user has access to.

#### Users Endpoint
Returns all users (requires admin privileges).

#### User Endpoint
Returns information about the currently authenticated user.

## Testing

A basic test playbook is available in the tests directory:

```bash
ansible-playbook tests/test.yml
```

## Troubleshooting

### Common Issues

| Status Code | Error | Solution |
|-------------|-------|----------|
| 401 | Unauthorized | Check your API token or username/password credentials |
| 403 | Forbidden | Verify the user has permission to access this resource |
| 404 | Not Found | Check that project_id and resource_id exist |
| 429 | Too Many Requests | Increase `semaphore_api_retry_delay` or reduce request frequency |
| 5xx | Server Error | Check Semaphore server logs for details |

### SSL Certificate Issues

For self-signed certificates (development only):

```yaml
vars:
  semaphore_api_validate_certs: false
```

### Debug Mode

Enable verbose output by running with verbose flags:

```bash
ansible-playbook your-playbook.yml -vvv
```

Or enable role-level debugging:

```yaml
vars:
  semaphore_api_debug: true
  semaphore_api_verbose_debug: true
```

## License

MIT

## Author Information

This role was created for managing Semaphore UI API interactions in Ansible automation workflows.

## Contributing

Contributions are welcome! Please submit issues or pull requests for any improvements or bug fixes.
