# Changelog

## 0.2.0

- `pasadizo_tcp_tuning` (opt-in): BBR congestion control, fq, and no TCP slow start after idle on the host, persisted in `/etc/sysctl.d/60-pasadizo.conf` and `/etc/modules-load.d/pasadizo.conf`; `pasadizo_sysctl_extra` adds more sysctls. The collection now depends on `ansible.posix` and `community.general`.

## 0.1.0

- Role `pasadizo.pasadizo.pasadizo`: installs a Pasadizo release from its archive (downloaded on the controller by default and checked against `checksums.txt`), keeps every version beside a symlink, writes the configuration from data after `pasadizo check` accepts it, runs the service as an unprivileged user under a hardened systemd unit, and optionally installs an end-to-end watchdog.
