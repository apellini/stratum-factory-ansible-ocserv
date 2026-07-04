# Changelog

## [0.2.1] - 2026-07-04

### Fixed

- Remove `listen-clear-file = /var/run/ocserv-conn.socket` from `ocserv.conf.j2`.
  This directive was removed in ocserv 1.1.2 and causes `config file error` on
  startup, leaving ocserv in a failed state. The bug was pre-existing in the template
  but was first triggered by the D-INFRA-22 config redeploy + restart.

## [0.2.0] - 2026-07-03

### Fixed

- Wire up `ocserv_listen_interface` so `listen-host` is actually rendered in
  `/etc/ocserv/ocserv.conf` (D-INFRA-22). Previously the variable was declared but never
  referenced — ocserv silently bound all interfaces (0.0.0.0).
- Change the default from the broken `eth0` to `""` (auto-detect). The role now resolves
  the external NIC from `ansible_facts.default_ipv4.interface` and derives its IPv4
  address; that IP is rendered as `listen-host`, restricting VPN binds to the external
  NIC only. Ubuntu 24.04 on GCP uses `ens4`/`ens5`, not `eth0`/`eth1`.
- Add a fail-fast `assert` task so misconfiguration (unresolvable interface) surfaces
  immediately with a clear message rather than silently producing a broken config.

## [0.1.3] - 2026-07-03

### Fixed

- Replace `when: ansible_os_family == "Debian"` with `when: ansible_facts['os_family'] == "Debian"`
  in the apt cache-refresh task. Top-level `ansible_*` fact injection (`INJECT_FACTS_AS_VARS`)
  is deprecated in ansible-core and will be removed in 2.24. Using `ansible_facts['os_family']`
  is the correct forward-compatible form.

## [0.1.2] - 2026-07-03

### Fixed

- Add `ansible.builtin.apt` cache refresh (Debian family, `cache_valid_time: 3600`)
  before the `ocserv` install task. On a freshly-booted Ubuntu 24.04 GCE VM the apt
  index is empty for the `universe` component, causing "No package matching 'ocserv'
  is available". The `when: ansible_os_family == "Debian"` guard keeps the role
  distro-agnostic for the generic `package` install that follows.

## [0.1.0] - 2026-07-02

### Added

- Initial release: ocserv install, self-signed TLS cert, PAM auth, ocserv.conf template
