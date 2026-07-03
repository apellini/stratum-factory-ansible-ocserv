# stratum-factory-ansible-ocserv

Ansible role that installs and configures **ocserv** (OpenConnect SSL VPN server) on the
STRATUM dev bastion (Ubuntu 24.04).

Part of the **STRATUM Factory** layer — stateless, all input from variables, documented
for humans and RAG.

---

## What the role does

1. Installs the `ocserv` package.
2. Creates a private certificate directory (`/etc/ocserv/certs`, mode 0700).
3. Generates a 4096-bit RSA private key.
4. Generates a CSR and issues a self-signed X.509 certificate (via `community.crypto`).
5. Deploys `/etc/ocserv/ocserv.conf` from a Jinja2 template.
6. Enables and starts the `ocserv` systemd service.

---

## Requirements

- **Ansible** >= 2.15
- **Collection:** `community.crypto` must be installed before running the role:

  ```bash
  ansible-galaxy collection install community.crypto
  ```

- Target host: Ubuntu 24.04 with `systemd`
- `become: true` (tasks require root)

---

## Role variables

All variables have defaults in `defaults/main.yml`. Override as needed.

| Variable | Default | Description |
|---|---|---|
| `ocserv_port` | `443` | TCP/UDP port ocserv listens on |
| `ocserv_listen_interface` | `""` | External-facing NIC whose IPv4 becomes `listen-host`; `""` = auto-detect from default-route interface |
| `ocserv_vpn_network` | `172.16.10.0` | VPN pool network address assigned to clients |
| `ocserv_vpn_netmask` | `255.255.255.0` | Subnet mask for the VPN pool |
| `ocserv_pushed_routes` | `["10.100.0.0/255.255.255.0"]` | Routes pushed to VPN clients |
| `ocserv_dns` | `8.8.8.8` | DNS server pushed to VPN clients |
| `ocserv_cert_cn` | `stratum-bastion` | Common Name for the self-signed TLS certificate |
| `ocserv_cert_dir` | `/etc/ocserv/certs` | Directory for the certificate and key |
| `ocserv_cert_days` | `3650` | Certificate validity in days (~10 years) |
| `ocserv_max_clients` | `10` | Maximum simultaneous VPN sessions |
| `ocserv_max_same_client` | `2` | Maximum simultaneous sessions per client certificate |
| `ocserv_compression` | `true` | Enable tunnel data compression |

---

## Example playbook

```yaml
- name: Configure bastion VPN
  hosts: bastion
  become: true
  roles:
    - role: stratum-factory-ansible-ocserv
      vars:
        ocserv_cert_cn: "dev-bastion.stratum.dev"
        ocserv_pushed_routes:
          - "10.100.0.0/255.255.255.0"
```

---

## PAM authentication

Authentication is handled by PAM — connecting VPN users must be Linux system accounts
on the bastion. User accounts are **not** managed by this role; they are provisioned
separately by the `stratum-factory-ansible-bastion-users` role (or equivalent). This
role only configures ocserv to delegate auth to PAM.

---

## TLS certificate warning

Because a **self-signed certificate** is generated, VPN clients will display a TLS
warning on first connection. This is **expected in the dev environment**. Accept the
certificate or pre-import it into your client's trust store.

For production use, replace the generated certificate with one signed by a trusted CA
by placing your cert/key at the paths defined by `ocserv_cert_dir` before running the
role (the `community.crypto` tasks are idempotent and will skip generation if the files
already exist).

---

## Listen-host auto-detection

`ocserv_listen_interface` defaults to `""`, which triggers auto-detection: the role
resolves the interface from `ansible_facts.default_ipv4.interface` (the NIC carrying the
default route) and derives its IPv4 address. That IP is rendered as `listen-host` in
`ocserv.conf`, restricting VPN binds to the external NIC only.

On Ubuntu 24.04 GCP instances the external NIC is `ens4` (not `eth0` — predictable NIC
naming, D-INFRA-22). Auto-detection is name-agnostic and self-heals across image changes.

Set `ocserv_listen_interface` explicitly only when the default-route interface is not the
desired bind NIC, or when testing with molecule (container has a single `eth0`).

The role requires `gather_facts: true` (the Ansible default). It fails fast with a clear
message if the interface cannot be resolved.

---

## Network context (STRATUM dev bastion)

The bastion has two NICs:

| NIC | Kernel name | Side | Address |
|---|---|---|---|
| nic0 | `ens4` | External VPC (internet-gateway route) | public IP |
| nic1 | `ens5` | Internal VPC | 10.100.0.2 / 10.100.0.0/24 |

ocserv auto-detects `ens4` as the listen-host interface (default-route NIC). The pushed
route `10.100.0.0/255.255.255.0` lets VPN clients reach the internal STRATUM VPC directly.

---

## Testing with Molecule

```bash
pip install molecule molecule-plugins[docker]
ansible-galaxy collection install community.crypto
molecule test
```

---

## License

MIT

## Author

apellini — STRATUM Factory
