# Platforms: Linux & Windows

rystemd's home is **Linux** — that's where every feature lives. **Windows**
runs a compatibility port of the same manager + CLI + TUI stack, so the *shape*
of things is familiar even if the plumbing underneath differs.

## Linux

Everything in this book applies. Highlights the Linux build adds:

- **cgroup v2 supervision** — each service runs in its own subtree, so `stop`
  takes down the whole tree (even children that tried to escape their process
  group).
- **Socket activation** the systemd way — listeners passed to the service via
  `LISTEN_FDS`, so inetd-style handoff works.
- **D-Bus** — rystemd talks to the system/user bus and, when no real systemd is
  present, serves an `org.freedesktop.systemd1`-compatible surface so
  systemd-aware tooling can connect.
- **udev device tracking** — `.device` units discovered from the kernel so
  services can order after hardware.
- **The PID-1 boot path** — see [Booting the machine itself](pid1.md).

## Windows

Windows is a *compatibility port*: same unit language, same `rystemctl`
commands, backed by native Win32 primitives. Two ways to run the manager:

**Per-user** (no elevation needed):

```powershell
rystemd.exe daemon --user
rystemctl.exe --user start hello.service
```

User units live under `%LOCALAPPDATA%\rystemd\`. For example:

```ini
[Unit]
Description=Windows worker

[Service]
Type=simple
ExecStart=C:\Tools\worker.exe --serve
Restart=on-failure
```

**As a native Windows service** (elevated):

```powershell
rystemd.exe service install            # automatic start
# or: rystemd.exe service install --manual
sc.exe start rystemd
rystemctl.exe list-units
sc.exe stop rystemd
rystemd.exe service uninstall
```

System units live under `%ProgramData%\rystemd\`.

### What works differently

| | Windows |
| --- | --- |
| Process supervision | Win32 **Job Object** (kill-on-close for the whole tree) |
| Service types | `simple`, `exec`, `idle`, `oneshot` |
| Timers & targets | Yes |
| TCP socket trigger | Yes — launch-on-connection (no `LISTEN_FDS` handoff) |
| `MemoryMax=` / `TasksMax=` | Win32 Job Object limits |
| `forking` / `notify` / `dbus` types | Explicit error |
| `User=` / `Group=` | Explicit error |
| Unix sockets, cgroups, mounts, devices, boot path | Linux only |

The parts that aren't supported on Windows fail *explicitly* — they're reported
as errors, not silently ignored, so you're never guessing why something didn't
run.

Next: the straight story on what is and isn't supported —
[Compatibility & known issues](compatibility.md).