# PKI Manager Ansible Collection

Ansible roles for managing X.509 certificates via the PKI Manager API.

## Contents

- **roles/pki_manager** - Main role for managing CAs and certificates

## Quick Start

### 1. Configure Credentials

Create a vars file or set environment variables:

```yaml
# group_vars/all.yml
pki_api_url: "https://pki.example.com/api/v1"
pki_oidc_url: "https://iam.example.com/realms/pki/protocol/openid-connect/token"
pki_client_id: "ansible-pki"
pki_client_secret: "{{ vault_pki_client_secret }}"  # Use Ansible Vault!
```

Or use environment variables:

```bash
export PKI_API_URL="https://pki.example.com/api/v1"
export PKI_OIDC_URL="https://iam.example.com/realms/pki/protocol/openid-connect/token"
export PKI_CLIENT_ID="your-client-id"
export PKI_CLIENT_SECRET="your-client-secret"
```

### 2. Use the Role

```yaml
- name: Issue a certificate
  hosts: webservers
  tasks:
    - name: Issue server certificate
      ansible.builtin.include_role:
        name: pki_manager
      vars:
        pki_api_url: "{{ lookup('env', 'PKI_API_URL') }}"
        pki_oidc_url: "{{ lookup('env', 'PKI_OIDC_URL') }}"
        pki_client_id: "{{ lookup('env', 'PKI_CLIENT_ID') }}"
        pki_client_secret: "{{ lookup('env', 'PKI_CLIENT_SECRET') }}"
        pki_action: cert_issue
        pki_ca_id: "your-ca-id"
        pki_cert_cn: "{{ inventory_hostname }}"
        pki_cert_type: server
        pki_cert_dns_names:
          - "{{ inventory_hostname }}"
          - "{{ ansible_fqdn }}"
```

## Installation

### From This Repository

```bash
# Clone the repository
git clone https://github.com/oriolrius/pki-manager-role.git

# Add to your ansible.cfg
[defaults]
roles_path = /path/to/pki-manager-role/roles
```

### Using requirements.yml

```yaml
# requirements.yml
- src: https://github.com/oriolrius/pki-manager-role.git
  scm: git
  version: main
  name: pki_manager
```

```bash
ansible-galaxy install -r requirements.yml
```

## Available Actions

| Action | Description |
|--------|-------------|
| `auth_test` | Test authentication |
| `ca_list` | List Certificate Authorities |
| `ca_get` | Get CA details |
| `ca_create` | Create a new CA |
| `ca_revoke` | Revoke a CA |
| `ca_delete` | Delete a CA |
| `cert_list` | List certificates |
| `cert_get` | Get certificate details |
| `cert_issue` | Issue a new certificate |
| `cert_renew` | Renew a certificate |
| `cert_revoke` | Revoke a certificate |
| `cert_delete` | Delete a certificate |
| `cert_download` | Download a certificate |
| `stats` | Get PKI statistics |
| `expiring` | List expiring certificates |
| `search` | Search CAs and certificates |

## Examples

See [roles/pki_manager/README.md](roles/pki_manager/README.md) for detailed examples.

### Issue and Deploy Certificate

```yaml
- name: Deploy certificates to web servers
  hosts: webservers
  tasks:
    - name: Issue certificate
      ansible.builtin.include_role:
        name: pki_manager
      vars:
        pki_action: cert_issue
        pki_ca_id: "{{ ca_id }}"
        pki_cert_cn: "{{ inventory_hostname }}"
        pki_cert_type: server
        pki_cert_dns_names: ["{{ inventory_hostname }}"]

    - name: Download certificate
      ansible.builtin.include_role:
        name: pki_manager
      vars:
        pki_action: cert_download
        pki_cert_id: "{{ pki_cert_id }}"
        pki_download_dest: "/etc/ssl/certs/{{ inventory_hostname }}.pem"

    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

## Testing

```bash
cd ansible/roles/pki_manager
ansible-playbook -i tests/inventory tests/test.yml
```

## License

MIT

## Author

Oriol Rius - [joor.net](https://joor.net)
