# Compatibility & known issues

rystemd understands the systemd unit language and lifecycle, but it is a
reimplementation — a few areas are deliberately partial. The full, current
matrix lives in
[`KNOWN_ISSUES.md`](https://github.com/rystemd/rystemd/blob/main/KNOWN_ISSUES.md);
this page is the honest short version.

## Unit types

**Supported:** `.service`, `.socket`, `.timer`, `.target`, `.mount`, `.path`,
`.device`.

**Not yet:** `.slice` / `.scope` (the cgroup hierarchy — `user.slice`,
`app.slice`, `session.slice`) and `.automount`. This is the main gap for a
strict drop-in on a modern desktop, which leans on slices.

## Hardening directives

Many sandbox directives are parsed and **enforced**: the mount-namespace set
(`PrivateTmp=`, `PrivateDevices=`, `ProtectHome=`, `ProtectSystem=`,
`ReadOnlyPaths=`), `NoNewPrivileges=`, `CapabilityBoundingSet=`,
`AmbientCapabilities=`, and the seccomp set (`SystemCallFilter=`,
`RestrictRealtime=`, `LockPersonality=`, `RestrictSUIDSGID=`,
`RestrictAddressFamilies=`, `MemoryDenyWriteExecute=`).

A handful are **parsed but not yet enforced** — they load with a warning rather
than failing, so your service still runs (just without that isolation):
`ProtectKernelModules=`, `DeviceAllow=`, `RestrictNamespaces=`, and others.

## Conditions & assertions

A working set of `Condition*=`. A few less-common kinds
(`ConditionVirtualization`, `ConditionNeedsUpdate`, `ConditionSecurity`,
`ConditionPathIsManifest`, `ConditionKernelCommandLine`,
`ConditionCredential`) are parsed but always-true/always-false, so units keyed
on them will always or never load.

## Everything else, briefly

- **Timeouts & limits:** `TimeoutStopFailureMode=` (used widely on Fedora),
  `TimeoutSec=`, and a few `LimitEXT*` relaxations aren't fully honored. Some
  desktop services (`user@1000.service`) are heavy on `PAMName`,
  `KeyringMode`, `Delegate`, `ImportCredential`, and `RequiresMountsFor=`,
  which aren't implemented yet.
- **Restart / readiness:** `Restart=` policies are implemented; the restart
  rate limit uses a fixed 5-in-10s window.

For the complete, version-specific matrix — every directive, its status, and
which real Fedora units it affects — see
[`KNOWN_ISSUES.md`](https://github.com/rystemd/rystemd/blob/main/KNOWN_ISSUES.md).