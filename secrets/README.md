# Secrets Directory

This directory contains encrypted Ansible vault files for storing sensitive data such as API tokens.

## Setup

### 1. Create a Vault Password

Choose a secure password and store it in a `.vault_pass` file (in the project root, not this directory):

```bash
echo "your_secure_password" > .vault_pass
chmod 600 .vault_pass
```

### 2. Set Environment Variable (Optional)

Instead of using a password file, you can set an environment variable:

```bash
export ANSIBLE_VAULT_PASSWORD="your_secure_password"
```

Or for persistent configuration, add to your shell profile (`~/.bashrc`, `~/.zshrc`):

```bash
export ANSIBLE_VAULT_PASSWORD="your_secure_password"
```

### 3. Create the Vault File

Using password file:
```bash
ansible-vault create secrets/vault.yml --vault-password-file .vault_pass
```

Using environment variable:
```bash
echo "your_secure_password" | ansible-vault create secrets/vault.yml --vault-password-file /dev/stdin
```

Add the following initial content when prompted:
```yaml
---
semaphore_api_token: ""
```

### 4. Configure the Semaphore Role

Enable token persistence in your playbook:

```yaml
- hosts: localhost
  roles:
    - role: semaphore_ui
      semaphore_admin_password: "your_admin_password"
      semaphore_save_api_token: true
      semaphore_vault_password_file: ".vault_pass"  # or omit if using env var
```

## Managing the Vault

### View Contents

```bash
ansible-vault view secrets/vault.yml --vault-password-file .vault_pass
```

### Edit Contents

```bash
ansible-vault edit secrets/vault.yml --vault-password-file .vault_pass
```

### Decrypt (temporarily)

```bash
ansible-vault decrypt secrets/vault.yml --vault-password-file .vault_pass
```

### Re-encrypt

```bash
ansible-vault encrypt secrets/vault.yml --vault-password-file .vault_pass
```

## Using the Token

In other playbooks, include the vault file to access the stored token:

```yaml
- hosts: localhost
  vars_files:
    - secrets/vault.yml
  tasks:
    - name: Call Semaphore API
      ansible.builtin.uri:
        url: "http://localhost:3000/api/projects"
        headers:
          Authorization: "Bearer {{ semaphore_api_token }}"
```

Run with:
```bash
ansible-playbook playbook.yml --vault-password-file .vault_pass
```

## Security Notes

- Never commit `.vault_pass` or plain-text passwords to version control
- The `secrets/` directory is configured in `.gitignore` to:
  - Ignore plain-text files (`secrets/*`)
  - Allow encrypted vault files (`!secrets/*.yml`)
- Use strong, unique passwords for vault encryption
- Consider using a secrets manager for the vault password in production
