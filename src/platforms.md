# Platforms

## Linux

Linux provides the complete feature set:

- cgroup v2 process supervision
- process-group fallback
- socket activation through `LISTEN_FDS` and `LISTEN_PID`
- partial D-Bus compatibility
- runtime device units from udev and netlink
- service sandboxing
- PID 1 boot support

See [Running as PID 1](pid1.md) before changing a boot configuration.

## Windows

Windows uses the same unit language and clients with a smaller manager feature
set.

User manager:

```powershell
rystemd.exe daemon --user
rystemctl.exe --user start hello.service
```

User units live under `%LOCALAPPDATA%\rystemd\`.

Windows service manager:

```powershell
rystemd.exe service install
sc.exe start rystemd
rystemctl.exe list-units
sc.exe stop rystemd
rystemd.exe service uninstall
```

System units live under `%ProgramData%\rystemd\`.

| Function | Windows behavior |
| --- | --- |
| Process supervision | Win32 Job Objects |
| Service types | `simple`, `exec`, `idle`, `oneshot` |
| Timers and targets | Supported |
| TCP socket trigger | Launch on connection, without `LISTEN_FDS` |
| `MemoryMax=` and `TasksMax=` | Win32 Job Object limits |
| `forking`, `notify`, `dbus` | Unsupported |
| `User=` and `Group=` | Unsupported |
| Mount, device, cgroup, PID 1 | Linux only |

Unsupported Windows settings return errors.
