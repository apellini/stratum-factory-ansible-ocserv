# Changelog

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
