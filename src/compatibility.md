# Compatibility

rystemd implements a subset of systemd behavior. The current detailed list is
maintained in
[`KNOWN_ISSUES.md`](https://github.com/rystemd/rystemd/blob/main/KNOWN_ISSUES.md).

## Unit types

Supported: `.service`, `.socket`, `.timer`, `.target`, `.mount`, `.path`, and
runtime `.device` units.

Unsupported: `.slice`, `.scope`, and `.automount` units.

## Transactions

Start and stop operations expose job IDs and completion results. Normal job
replacement and `replace-irreversibly` are implemented for tested service
conflicts. Full transaction merging, cycle diagnostics, all job modes, and the
D-Bus job object model remain incomplete.

## Sandboxing

Implemented directives include `PrivateTmp=`, `PrivateDevices=`,
`ProtectHome=`, `ProtectSystem=`, `ReadOnlyPaths=`, `NoNewPrivileges=`,
`CapabilityBoundingSet=`, `AmbientCapabilities=`, and `SystemCallFilter=`.

Some parsed directives warn without enforcement. Examples include
`ProtectKernelModules=`, `DeviceAllow=`, and `RestrictNamespaces=`.

## Control surfaces

The native client uses a local socket. D-Bus support covers selected manager and
unit operations. It does not provide complete systemd1 job objects, signals, or
property-change behavior.

The journal uses a rystemd-specific on-disk format. Native journald protocol
compatibility is not implemented. `systemd-run`, `varlinkctl`, and several less
common `systemctl` operations are unavailable.

## Platform limits

Linux is the primary target. Windows omits Unix process, identity, socket,
mount, cgroup, and PID 1 behavior. Static musl builds do not provide host glibc
NSS integration.
