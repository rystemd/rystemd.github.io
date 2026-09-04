# Introduction

rystemd is a systemd-compatible service manager written in Rust.

It contains three programs:

- `rystemd`: unit manager and daemon
- `rystemctl`: command-line client
- `rystemd-tui`: terminal client

Supported unit types include services, sockets, timers, targets, mounts, paths,
and runtime device units. Linux is the primary platform. Windows supports a
smaller service-manager subset.

Compatibility is selective. Unsupported behavior is listed in
[Compatibility](compatibility.md) and in the repository
[`KNOWN_ISSUES.md`](https://github.com/rystemd/rystemd/blob/main/KNOWN_ISSUES.md).

Start with [Installing](install.md), then [Getting started](getting-started.md).
PID 1 use is covered separately in [Running as PID 1](pid1.md).
