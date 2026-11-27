# Ansible collection for Pasadizo

The collection `pasadizo.pasadizo` installs and runs [Pasadizo](https://github.com/pasadizo-net/pasadizo) on Linux hosts. It has one role, `pasadizo.pasadizo.pasadizo`.

The role:

- downloads a release archive on the controller (or on the host) and checks it against the release's `checksums.txt`, so a host on a filtered network that cannot reach GitHub still installs;
- keeps every version in `/usr/local/lib/pasadizo/pasadizo-<version>` and points `/usr/local/bin/pasadizo` at the current one, so a rollback is a version change and a restart;
- writes the configuration from data (`pasadizo_config`) as JSON, and replaces the file on the host only after `pasadizo check` accepts it;
- runs the service as the unprivileged user `pasadizo` under a hardened systemd unit, with only the capabilities you list;
- can install a watchdog that fetches URLs through each proxy and restarts the service when a proxy accepts connections but cannot carry them.

## Installation

```yaml
# requirements.yml
collections:
  - name: https://github.com/pasadizo-net/ansible.git
    type: git
    version: v0.3.0
```

```sh
ansible-galaxy collection install -r requirements.yml
```

## Example

```yaml
- hosts: gateways
  become: true
  roles:
    - role: pasadizo.pasadizo.pasadizo
      vars:
        pasadizo_version: "1.15.0-alpha.14-pasadizo.7"
        pasadizo_watchdog_enabled: true
        pasadizo_watchdog_proxies: ["http://127.0.0.1:2080"]
        pasadizo_config:
          log: {level: warn}
          inbounds:
            - {type: mixed, tag: in, listen: "::", listen_port: 2080}
          outbounds:
            - {type: direct, tag: direct}
          route: {final: direct}
```

## Variables

All variables are in [`roles/pasadizo/defaults/main.yml`](roles/pasadizo/defaults/main.yml) and are checked against [`roles/pasadizo/meta/argument_specs.yml`](roles/pasadizo/meta/argument_specs.yml). The ones you usually set:

| Variable | Default | What |
| --- | --- | --- |
| `pasadizo_config` | (required) | The configuration, as data. |
| `pasadizo_version` | the latest release the collection was tested with | Release to install, without the leading `v`. |
| `pasadizo_service_state` | `started` | `stopped` installs and checks everything without binding a port, for a staged cutover. |
| `pasadizo_libcronet` | `true` | Install `libcronet.so` beside the binary, for the NaiveProxy outbound. |
| `pasadizo_capabilities` | `[]` | For example `CAP_NET_BIND_SERVICE` for ports below 1024, `CAP_NET_ADMIN` for `tun`. |
| `pasadizo_config_diff` | `true` | Turn it off when the configuration holds secrets that must not appear in `--diff` output. |
| `pasadizo_watchdog_enabled` | `false` | End-to-end health check through `pasadizo_watchdog_proxies`. |
| `pasadizo_tcp_tuning` | `false` | BBR, fq and no slow start after idle for the whole host, persisted across reboots. |

Tags: `pasadizo-install`, `pasadizo-config`, `pasadizo-watchdog`, `pasadizo-tuning`. A run with `-t pasadizo-config` writes and checks the configuration and restarts the service when it changed.

## Testing

CI runs `ansible-lint` and installs the collection on GitHub's amd64 and arm64 Ubuntu runners with [`tests/playbook.yml`](tests/playbook.yml): it fetches a page through the proxy, runs the watchdog, checks that the service does not run as root and that an invalid configuration is refused without replacing the old one, then runs again and requires no change.

## License

GPL-3.0-or-later, like Pasadizo. Pasadizo is derived from sing-box and is not affiliated with it or its authors.
