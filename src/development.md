# Development

This page is for people working *on* rystemd — the crate layout, how the
manager is built, the platform internals, and the release process. If you're a
user (writing units, running the CLI), you don't need any of this.

## Workspace layout

Three crates in one Rust workspace:

- **`rystemd`** — the library + daemon binary. Everything else builds on it.
- **`rystemctl`** — the `systemctl`-compatible CLI.
- **`rystemd-tui`** — the terminal client.

## The manager

`Manager` drives a synchronous, single-threaded poll loop — **no async runtime,
no tokio**. The loop iterates on a bounded timeout, draining:

- control requests over the IPC channel,
- D-Bus requests, events, and unit snapshots (Linux, `dbus` feature),
- timer-wheel elapses (`fire_timer`),
- path-watch polls (`.path` units),
- SIGCHLD reaps and job resolution (`process_jobs`).

Jobs (start/stop/restart) encode the lifecycle transition and the `Wants=`,
`Requires=`, `After=`, `Before=`, `Conflicts=` ordering graph. A per-unit-type
VTable (`unit_type` → `UnitType` trait) handles the concrete `start`/`stop` for
each kind.

### Unit types

| Kind | Dir/file | Notes |
| --- | --- | --- |
| `.service` | `(suffix)` | `Type=` simple/exec/idle/oneshot/forking/notify/dbus |
| `.socket` | `[Socket]` | `ListenStream=/Datagram=/Netlink=/SequentialPacket=`, inetd-style activation |
| `.timer` | `[Timer]` | calendar + monotonic triggers |
| `.target` | — | grouping only |
| `.mount` | `[Mount]` | `mount(2)` / `umount(2)` |
| `.path` | `[Path]` | `PathExists=`, `DirectoryNotEmpty=`, `PathChanged=`, `PathExistsGlob=` |
| `.device` | udev | runtime device units (Linux) |

The parser lives in `unit/` with a `parse::RawUnitFile` layer; directives are
pulled via `unit_scalar`/`list_of` helpers so unsupported directives fail loudly
or are parsed-and-warned, never silently lost.

### repo

A typed DAO (`UnitDefinition`, not unit-file text) over a directory backend.
Mutations are atomic (temp + rename); reads are never locked. Single process by
design. `rystemctl repo` and the manager share it so clients can inspect or edit
unit files in place.

### Control

`Control` is a single interface with two implementations: the in-process
`Manager` and the remote `SocketClient`. The manager also exposes the same
surface over JSON IPC and (Linux, `dbus`) over D-Bus. This is what `rystemctl`,
`rystemd-tui`, and D-Bus clients speak.

## Platform layer

`platform/` abstracts the OS. Linux uses `nix`; Windows uses direct Win32
bindings. The manager never touches raw syscalls directly — everything is
behind this layer (process spawn + cgroups/Job Objects, sandboxing mounts,
signal handling, udev/netlink, D-Bus, boot).

### Linux internals

- **Process supervision.** A service runs in its own **cgroup v2** subtree; the
  spawned process adopts itself into the unit's cgroup before exec, so
  `cgroup.kill` reaches every descendant. When cgroups aren't usable the
  fallback is a **process group**: the child becomes a session leader so
  `kill(-pid, sig)` reaches the tree, and it can acquire a controlling
  terminal. As PID 1 (or with the daemon running as root) rystemd sets itself
  as a subreaper and sweeps orphaned/double-forked children.
- **Socket activation.** Listeners are passed to the child via `LISTEN_FDS` /
  `LISTEN_PID` (the standard systemd contract). `systemd-udevd` sockets,
  journald, and coredump all rely on this; without it udev receives no kernel
  events.
- **Path activation** is stat-based (polled in the manager's loop; no inotify
  yet).
- **`.device` units** are discovered from sysfs and tracked via netlink
  uevents (`udev` feature).
- **D-Bus.** With `dbus`, rystemd owns `org.rystemd.Manager1` and — when the
  well-known name is free, i.e. no real systemd is present — also serves an
  `org.freedesktop.systemd1`-compatible surface: `Version`, `ListUnits`,
  `GetUnit`/`LoadUnit`, `ListJobs`, `GetUnitProcesses`, per-unit
  `org.freedesktop.systemd1.Unit` objects, and `UnitNew`/`UnitRemoved`. The
  systemd1 surface uses systemd's `bus_label_escape` object-path encoding.

### The PID-1 boot path & real-root handoff

With the `boot` feature, rystemd can run as PID 1: it mounts the API/virtual
filesystems, runs early-boot configuration (hostname, machine-id, sysctl,
modules, random-seed, tmpfiles dirs, `/etc/fstab`), boots the default target,
and powers off on shutdown.

**Real-root handoff (initramfs → deployment).** When launched as the stage-2
init inside an initramfs, rystemd pivots out of the throwaway rootfs into a real
deployment **before** reading any config — the classic `switch_root(8)`
sequence:

1. `in_initramfs()` detects an initramfs (`/` is a `rootfs`/`tmpfs`/`ramfs`
   mount, read from `/proc/self/mounts`).
2. `mount_sysroot_from_cmdline()` best-effort mounts the real root at `/sysroot`
   from `root=`/`rootfstype=`/`rootflags=` when nothing else has — so rystemd
   can be *its own* initramfs init (Model B), not only a post-pivot init
   (Model A). If `/sysroot` is already staged (ostree/dracut mounted it) it's a
   no-op.
3. `find_deployment()` resolves the *actual deployment* under `/sysroot`: on a
   plain root that *is* the sysroot; on an ostree sysroot it is
   `ostree/deploy/<os>/deploy/<commit>` (newest such dir with `usr`+`etc`).
4. `prepare_deployment()` binds the shared `/var` (which on ostree lives outside
   the deployment, at the sysroot) into the deployment.
5. `handoff(&deploy)` performs:
   - bind-mount `deploy` onto itself (promote it to a mountpoint — MS_MOVE
     requires a mount source),
   - `chdir(deploy)` → `mount(".", "/", MS_MOVE)` → `chroot(".")`
     → `chdir("/")` → re-`exec` the manager against the real root.

If the conditions are absent (`--user`, container, or the self-contained
initramfs harness), the handoff is skipped and rystemd boots in place — so the
normal harness flows are unchanged. The e2e is `scripts/test-handoff.sh`, which
now stages the deployment in a **real ostree layout** (under
`ostree/deploy/fedora/deploy/<commit>` with a sysroot-level `/var`) so discovery,
`/var` prep, and the bind-promote-then-MOVE path are all exercised. Note two
`switch_root` gotchas the test surfaces: MS_MOVE needs the source to be a
mountpoint (hence the self-bind), and the re-exec must NOT append an empty argv
entry — execv terminates on the null *pointer*, and an empty string would be
parsed as a CLI arg.

**Still open:** SELinux policy labeling (a foundation module ships in `pol/`;
live enforcing iteration remains), and launching a graphical session / display
manager. Validate a handoff boot on a throwaway VM before trusting it on a
primary install. A full real-VM run on an actual ostree image is UI-tested by
the scripts in [`docs/real-host-boot-vm.md`](https://github.com/rystemd/rystemd/blob/main/docs/real-host-boot-vm.md)
(`prepare-realinit-vm.sh` + `boot-realinit-vm.sh`), which need a rootful Fedora
host with libguestfs.

### Windows internals

A compatibility port. The manager, CLI, and TUI run with systemd-like semantics
backed by Win32 primitives: named pipes for IPC, Win32 **Job Objects** for
process supervision and resource limits, and Service Control Manager hosting for
system mode.

## Environment overrides (testing)

`RYSTEMD_UNIT_PATH`, `RYSTEMD_CONFIG_DIR`, `RYSTEMD_RUNTIME_DIR`,
`RYSTEMD_SOCKET`, and `RYSTEMD_JOURNAL_DIR` repoint the manager at a scratch
tree so integration tests run the real daemon against an isolated layout.

## Build & test

```sh
cargo build                 # debug (default features: socket, udev, dbus)
cargo build --features boot # add the PID-1 boot path
cargo test --workspace      # unit + integration
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all --check
```

The release profile is tuned for small, audit-friendly binaries:
`opt-level = "z"`, `lto = "fat"`, `codegen-units = 1`, `panic = "abort"`,
`strip = true`. The D-Bus integration test (`rystemd/tests/dbus.rs`) spins up a
private `dbus-daemon` and exercises the `org.freedesktop.systemd1` surface
end-to-end. Privileged tests (`rystemd/tests/privileged.rs`) cover `User=`
privilege dropping and `PrivateTmp=` mount namespaces and need real root (CI
runs them via `sudo`). The PID-1 boot surface is covered by unit tests plus VM
harnesses (`live-vm.sh`, `vm-test.sh`, `test-handoff.sh`) that boot a real
initramfs in qemu.

## Releases

Versioning follows the Git tag via `build.rs` (the tag is the source of truth;
no version string is hardcoded). Tag and push with:

```sh
./release.sh patch          # prompts before pushing
./release.sh patch -y       # tag + push in one step
```

The GitHub Actions release workflow builds Linux (`x86_64`, `aarch64`) and
Windows packages, runs the full test gate (including the D-Bus and privileged
suites), and attaches the artifacts to a GitHub Release. Linux: `tar.gz`,
`.deb`, `.rpm`. Windows: portable zip, MSI, and a NuGet package.

The release tarballs are consumed by the Homebrew tap (`rystemd/homebrew-rystemd`),
the Scoop bucket (`rystemd/scoop-rystemd`), and the NuGet package. Refresh the
Homebrew formula's pinned sha256 per release with
`scripts/gen-brew-formula.sh <version>` in the tap.