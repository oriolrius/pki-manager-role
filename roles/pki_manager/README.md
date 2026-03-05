# PKI Manager Ansible Role

Manage X.509 certificates via the PKI Manager API using Ansible.

## Requirements

- Ansible >= 2.14
- Access to a PKI Manager instance with OIDC authentication
- OIDC client credentials (client_id and client_secret)

## Role Variables

### Required Variables

| Variable | Description |
|----------|-------------|
| `pki_api_url` | PKI Manager API URL (e.g., `https://pki.example.com/api/v1`) |
| `pki_oidc_url` | OIDC token endpoint URL |
| `pki_client_id` | OIDC client ID |
| `pki_client_secret` | OIDC client secret |
| `pki_action` | Action to perform (see Actions below) |

### Default Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `pki_token_cache_file` | `/tmp/.pki_manager_token` | Token cache file path |
| `pki_token_refresh_margin` | `60` | Refresh token if less than N seconds remaining |
| `pki_cert_validity_days` | `365` | Default certificate validity |
| `pki_cert_algorithm` | `RSA-2048` | Default certificate algorithm |
| `pki_cert_type` | `server` | Default certificate type |
| `pki_ca_validity_days` | `3650` | Default CA validity |
| `pki_ca_algorithm` | `RSA-4096` | Default CA algorithm |
| `pki_api_timeout` | `30` | API timeout in seconds |
| `pki_output_dir` | `/etc/pki/certs` | Default certificate output directory |
| `pki_validate_certs` | `true` | Validate SSL certificates |
| `pki_quiet` | `false` | Suppress debug output |

## Actions

### Authentication

| Action | Description | Required Variables |
|--------|-------------|-------------------|
| `auth_test` | Test authentication | - |

### Certificate Authorities

| Action | Description | Required Variables |
|--------|-------------|-------------------|
| `ca_list` | List all CAs | - |
| `ca_get` | Get CA details | `pki_ca_id` |
| `ca_create` | Create a new CA | `pki_ca_cn`, `pki_ca_org`, `pki_ca_country` |
| `ca_revoke` | Revoke a CA | `pki_ca_id` |
| `ca_delete` | Delete a CA | `pki_ca_id` |

### Certificates

| Action | Description | Required Variables |
|--------|-------------|-------------------|
| `cert_list` | List certificates | - |
| `cert_get` | Get certificate details | `pki_cert_id` |
| `cert_issue` | Issue a new certificate | `pki_ca_id`, `pki_cert_cn` |
| `cert_renew` | Renew a certificate | `pki_cert_id` |
| `cert_revoke` | Revoke a certificate | `pki_cert_id` |
| `cert_delete` | Delete a certificate | `pki_cert_id` |
| `cert_download` | Download a certificate | `pki_cert_id` |

### Dashboard

| Action | Description | Required Variables |
|--------|-------------|-------------------|
| `stats` | Get PKI statistics | - |
| `expiring` | List expiring certificates | - |
| `search` | Search CAs and certificates | `pki_search_query` |

## Action-Specific Variables

### CA Creation (`ca_create`)

| Variable | Required | Description |
|----------|----------|-------------|
| `pki_ca_cn` | Yes | Common Name |
| `pki_ca_org` | Yes | Organization |
| `pki_ca_country` | Yes | Country (2-letter code) |
| `pki_ca_ou` | No | Organizational Unit |
| `pki_ca_state` | No | State/Province |
| `pki_ca_locality` | No | City/Locality |
| `pki_ca_algorithm` | No | Key algorithm |
| `pki_ca_validity` | No | Validity in days |

### Certificate Issuance (`cert_issue`)

| Variable | Required | Description |
|----------|----------|-------------|
| `pki_ca_id` | Yes | Issuing CA ID |
| `pki_cert_cn` | Yes | Common Name |
| `pki_cert_type` | No | Type: `server`, `client`, `email`, `code_signing` |
| `pki_cert_org` | No | Organization |
| `pki_cert_country` | No | Country (2-letter code) |
| `pki_cert_ou` | No | Organizational Unit |
| `pki_cert_dns_names` | No | List of DNS SANs |
| `pki_cert_ip_addresses` | No | List of IP SANs |
| `pki_cert_emails` | No | List of email SANs |
| `pki_cert_algorithm` | No | Key algorithm |
| `pki_cert_validity` | No | Validity in days |

### Certificate Download (`cert_download`)

| Variable | Required | Description |
|----------|----------|-------------|
| `pki_cert_id` | Yes | Certificate ID |
| `pki_download_format` | No | Format: `pem`, `der`, `pkcs12`, `jks` |
| `pki_download_password` | For PKCS12/JKS | Export password |
| `pki_download_dest` | No | Destination file path |
| `pki_download_mode` | No | File permissions (default: `0600`) |
| `pki_download_owner` | No | File owner |
| `pki_download_group` | No | File group |

### Revocation (`ca_revoke`, `cert_revoke`)

| Variable | Required | Description |
|----------|----------|-------------|
| `pki_revocation_reason` | No | Reason (see valid reasons below) |

Valid revocation reasons:
- `unspecified`
- `keyCompromise`
- `caCompromise`
- `affiliationChanged`
- `superseded`
- `cessationOfOperation`
- `certificateHold`
- `removeFromCRL`
- `privilegeWithdrawn`
- `aaCompromise`

## Output Variables

After each action, the role sets fact variables with the results:

| Action | Output Variables |
|--------|------------------|
| `ca_list` | `pki_cas`, `pki_ca_count` |
| `ca_get` | `pki_ca` |
| `ca_create` | `pki_ca`, `pki_ca_id` |
| `cert_list` | `pki_certificates`, `pki_cert_count` |
| `cert_get` | `pki_certificate` |
| `cert_issue` | `pki_certificate`, `pki_cert_id` |
| `cert_renew` | `pki_certificate`, `pki_new_cert_id` |
| `cert_download` | `pki_downloaded_file` |
| `stats` | `pki_stats` |
| `expiring` | `pki_expiring_certs`, `pki_expiring_count` |
| `search` | `pki_search_results`, `pki_search_count` |

## Examples

### Basic Configuration

```yaml
# group_vars/all.yml or playbook vars
pki_api_url: "https://pki.example.com/api/v1"
pki_oidc_url: "https://iam.example.com/realms/pki/protocol/openid-connect/token"
pki_client_id: "ansible-pki"
pki_client_secret: "{{ vault_pki_client_secret }}"
```

### List Certificate Authorities

```yaml
- name: List all CAs
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: ca_list

- name: Display CAs
  ansible.builtin.debug:
    var: pki_cas
```

### Create a CA and Issue a Certificate

```yaml
- name: Create a CA
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: ca_create
    pki_ca_cn: "Internal CA"
    pki_ca_org: "My Organization"
    pki_ca_country: "ES"
    pki_ca_algorithm: "RSA-4096"
    pki_ca_validity: 3650

- name: Store CA ID
  ansible.builtin.set_fact:
    my_ca_id: "{{ pki_ca_id }}"

- name: Issue a server certificate
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: cert_issue
    pki_ca_id: "{{ my_ca_id }}"
    pki_cert_cn: "web.example.com"
    pki_cert_type: "server"
    pki_cert_dns_names:
      - "web.example.com"
      - "www.example.com"
    pki_cert_ip_addresses:
      - "192.168.1.100"
    pki_cert_validity: 365
```

### Download Certificate for Nginx

```yaml
- name: Download certificate as PEM
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: cert_download
    pki_cert_id: "{{ my_cert_id }}"
    pki_download_format: pem
    pki_download_dest: "/etc/nginx/ssl/web.example.com.pem"
    pki_download_mode: "0640"
    pki_download_owner: root
    pki_download_group: nginx
```

### Download Certificate for Java (PKCS12/JKS)

```yaml
- name: Download certificate as PKCS12
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: cert_download
    pki_cert_id: "{{ my_cert_id }}"
    pki_download_format: pkcs12
    pki_download_password: "{{ vault_keystore_password }}"
    pki_download_dest: "/opt/app/keystore.p12"

- name: Download certificate as JKS
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: cert_download
    pki_cert_id: "{{ my_cert_id }}"
    pki_download_format: jks
    pki_download_password: "{{ vault_keystore_password }}"
    pki_download_dest: "/opt/app/keystore.jks"
```

### Renew Expiring Certificates

```yaml
- name: Get expiring certificates
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: expiring
    pki_expiring_limit: 50

- name: Renew each expiring certificate
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: cert_renew
    pki_cert_id: "{{ item.id }}"
    pki_cert_validity: 365
  loop: "{{ pki_expiring_certs }}"
  loop_control:
    label: "{{ item.cn | default(item.id) }}"
  when: item.daysRemaining | int < 30
```

### Revoke a Compromised Certificate

```yaml
- name: Revoke compromised certificate
  ansible.builtin.include_role:
    name: pki_manager
  vars:
    pki_action: cert_revoke
    pki_cert_id: "{{ compromised_cert_id }}"
    pki_revocation_reason: "keyCompromise"
```

## Testing

Run the test playbook:

```bash
# Set environment variables
export PKI_API_URL="https://pki.example.com/api/v1"
export PKI_OIDC_URL="https://iam.example.com/realms/pki/protocol/openid-connect/token"
export PKI_CLIENT_ID="your-client-id"
export PKI_CLIENT_SECRET="your-client-secret"

# Run tests
cd roles/pki_manager
ansible-playbook -i tests/inventory tests/test.yml
```

## License

MIT

## Author Information

Oriol Rius - [joor.net](https://joor.net)
