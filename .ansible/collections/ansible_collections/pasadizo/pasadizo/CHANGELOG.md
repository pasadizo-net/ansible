# Changelog

## 0.1.0

- Role `pasadizo.pasadizo.pasadizo`: installs a Pasadizo release from its archive (downloaded on the controller by default and checked against `checksums.txt`), keeps every version beside a symlink, writes the configuration from data after `pasadizo check` accepts it, runs the service as an unprivileged user under a hardened systemd unit, and optionally installs an end-to-end watchdog.
