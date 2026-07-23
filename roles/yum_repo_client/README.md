# Role: `yum_repo_client`

Configures yum/dnf repository entries on RHEL client hosts by looping a
caller-supplied list of dicts through `ansible.builtin.yum_repository`.
Each list entry is passed through to the module as-is (1:1 with module
arguments).

This role does **not** host a repository (ISO mount, httpd, vsftpd). That
concern belongs in a separate server role if needed.

## Requirements

- ansible-core 2.15+
- Supported platforms: **RHEL 8, 9, and 10**

## Role Variables

These are the only variables this role defines. Supply them via play
`vars`, extra vars, survey, or inventory/`group_vars`.

| Variable | Default | Description |
|----------|---------|-------------|
| `yum_repo_client_repositories` | `[]` | List of repo dicts passed 1:1 to `yum_repository`. Empty list is a successful no-op. |
| `yum_repo_client_clean_metadata` | `true` | After configure, run `dnf`/`yum clean all` (`changed_when: false`). |
| `yum_repo_client_verify` | `true` | After configure, assert each enabled present repo id appears as a leading column in `repolist --enabled`. Runs with `check_mode: false` so `--check` still validates against the live repolist. |

There are no role-only keys on repository dicts today. Every key on each
entry must be a valid `ansible.builtin.yum_repository` argument; unknown
keys cause the module to fail.

### Per-repository keys (role-enforced)

Required when `state` is `present` (the default):

| Key | Notes |
|-----|-------|
| `name` | Repo id (required by the module) |
| `description` | Human-readable description (role-required for present) |
| One of `baseurl`, `metalink`, or `mirrorlist` | Location (role-required for present) |

Required when `state` is `absent`: `name` only.

### Per-repository keys (module passthrough)

All `ansible.builtin.yum_repository` parameters are supported on each
list entry. Summary (see `ansible-doc ansible.builtin.yum_repository`
for choices and full descriptions):

| Parameter | Type | Notes |
|-----------|------|-------|
| `name` | str | Unique repository ID (required) |
| `state` | str | `present` (default) or `absent` |
| `description` | str | Human-readable repo name in the `.repo` file |
| `baseurl` | list/str | Repo base URL(s) |
| `metalink` | str | Metalink URL |
| `mirrorlist` | str | Mirrorlist URL |
| `file` | str | `.repo` basename (without extension); defaults to `name` |
| `reposdir` | path | Directory for `.repo` files (default `/etc/yum.repos.d`) |
| `enabled` | bool | Whether yum/dnf uses the repo (yum default: true) |
| `gpgcheck` | bool | Package GPG check |
| `repo_gpgcheck` | bool | Repodata GPG check |
| `gpgkey` | list/str | GPG key URL(s) |
| `gpgcakey` | str | CA key URL (deprecated upstream) |
| `username` | str | Basic auth username |
| `password` | str | Basic auth password (secret — see below) |
| `proxy` | str | Proxy URL, or `_none_` to disable global proxy |
| `proxy_username` | str | Proxy username |
| `proxy_password` | str | Proxy password (secret — see below) |
| `sslverify` | bool | Verify SSL certificates/hosts |
| `sslcacert` | str | CA certificate path/dir |
| `sslclientcert` | str | Client certificate path |
| `sslclientkey` | str | Client key path |
| `ssl_check_cert_permissions` | bool | Check cert path permissions |
| `priority` | str | Repo priority 1–99 (needs yum-plugin-priorities where applicable) |
| `cost` | str | Relative cost of accessing this repo |
| `exclude` | list | Packages to exclude |
| `includepkgs` | list | Packages to allow from this repo only |
| `include` | str | Include external config (path or URL) |
| `enablegroups` | bool | Allow package groups from this repo |
| `failovermethod` | str | `roundrobin` or `priority` |
| `http_caching` | str | `all`, `packages`, or `none` |
| `keepalive` | bool | HTTP/1.1 keepalive |
| `timeout` | str | Connection timeout (seconds) |
| `retries` | str | Download retry count |
| `throttle` | str | Bandwidth throttle |
| `bandwidth` | str | Max bandwidth with `throttle` |
| `metadata_expire` | str | Metadata expiry (seconds) |
| `metadata_expire_filter` | str | Filter for metadata expiry |
| `mirrorlist_expire` | str | Mirrorlist cache expiry |
| `ip_resolve` | str | `4`/`IPv4`, `6`/`IPv6`, or `whatever` |
| `skip_if_unavailable` | bool | Continue if repo is unreachable |
| `countme` | bool | Weekly metalink/mirrorlist countme flag |
| `module_hotfixes` | bool | Disable module RPM filtering |
| `protect` | bool | Protect packages (deprecated upstream) |
| `s3_enabled` | bool | S3 repo support (needs plugin) |
| `ui_repoid_vars` | str | Extra yum vars when displaying repo id |
| `async` | bool | Parallel metadata/package download when possible |
| `deltarpm_percentage` | str | Delta RPM size threshold |
| `deltarpm_metadata_percentage` | str | Delta RPM metadata size threshold |
| `owner` | str | File owner for the `.repo` file |
| `group` | str | File group for the `.repo` file |
| `mode` | raw | File mode for the `.repo` file |
| `attributes` | str | Filesystem attributes for the `.repo` file |
| `seuser` / `serole` / `setype` / `selevel` | str | SELinux context parts |
| `unsafe_writes` | bool | Allow non-atomic writes |

Secret values (`password`, `proxy_password`) must come from Ansible Vault
or AAP credentials — never commit them in defaults or example inventory.
When any entry defines those keys, the configure task enables `no_log`.

## Tags

| Tag | Scope |
|-----|-------|
| `yum_repo_client` | Entire role |
| `yum_repo_client_validate` | Asserts |
| `yum_repo_client_configure` | `yum_repository` loop |
| `yum_repo_client_clean` | Metadata clean |
| `yum_repo_client_verify` | Repolist check |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Configure yum/dnf repositories on clients
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: yum_repo_client
      vars:
        yum_repo_client_repositories:
          - name: company-baseos
            description: Company BaseOS mirror
            file: company-rhel
            baseurl: >-
              https://repos.example.com/rhel-{{ ansible_distribution_major_version }}/BaseOS/x86_64/os/
            enabled: true
            gpgcheck: true
            gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
            sslverify: true
            priority: "10"

          - name: company-appstream
            description: Company AppStream mirror
            file: company-rhel
            baseurl: >-
              https://repos.example.com/rhel-{{ ansible_distribution_major_version }}/AppStream/x86_64/os/
            enabled: true
            gpgcheck: true
            gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

          - name: epel
            description: EPEL
            metalink: >-
              https://mirrors.fedoraproject.org/metalink?repo=epel-{{ ansible_distribution_major_version }}&arch=$basearch
            enabled: false
            gpgcheck: true
            countme: true
```

Or use the thin entry-point playbook
`playbooks/linux-yum-repo-client.yml` and supply
`yum_repo_client_repositories` via survey, extra vars, or inventory
`group_vars`.

## Migration from `yum_repo_config`

| Old | New |
|-----|-----|
| Role `yum_repo_config` | `yum_repo_client` |
| Playbook `linux-yum-repo-config.yml` | `linux-yum-repo-client.yml` |
| JT `ACL_linux_yum_repo_config` | `ACL_linux_yum_repo_client` |
| `yum_repo_config_mode: client` + server `hostvars` baseurl | Explicit list of repo dicts with `baseurl` / `metalink` / `mirrorlist` |
| `yum_repo_config_mode: server` | Removed — track separately as `yum_repo_server` if still needed |

Example migration of a former client play that pointed at a lab mirror:

```yaml
# Before (implicit server discovery)
yum_repo_config_mode: client

# After (explicit URLs)
yum_repo_client_repositories:
  - name: remote-repo-baseos
    description: Remote Repository BaseOS
    file: remote_repo
    baseurl: http://repo.example.com/RHEL/BaseOS
    enabled: true
    gpgcheck: false
  - name: remote-repo-appstream
    description: Remote Repository AppStream
    file: remote_repo
    baseurl: http://repo.example.com/RHEL/AppStream
    enabled: true
    gpgcheck: false
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
