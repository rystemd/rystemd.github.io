# Platforms

## Feature matrix

`Partial` means that the feature exists with documented limits. `No` means no
supported implementation.

| Feature | Linux | Windows | macOS |
| --- | --- | --- | --- |
| Manager daemon | Yes | Yes | No |
| `rystemctl` client | Yes | Yes | No |
| `rystemd-tui` | Yes | Yes | No |
| Service units | Yes | Partial | No |
| Timer and target units | Yes | Yes | No |
| Socket activation | Yes | Partial | No |
| Path units | Yes | No | No |
| Mount units | Yes | No | No |
| Device units | Yes | No | No |
| Process-tree supervision | cgroup v2 or process groups | Win32 Job Objects | No |
| Resource limits | cgroup v2 and POSIX limits | Win32 Job Objects | No |
| Service sandboxing | Partial | No | No |
| `User=` and `Group=` | Yes | No | No |
| D-Bus integration | Partial | No | No |
| Native system-service integration | PID 1 | Windows Service Control Manager | No |
| PID 1 boot | Experimental | No | No |
| Release artifacts | GNU, musl, RPM, DEB, APK, tar | MSI, zip, Scoop, NuGet | No |

Windows service support covers `simple`, `exec`, `idle`, and `oneshot`.
Windows socket activation starts a service on TCP connection but does not pass
listeners through `LISTEN_FDS`.

macOS has no tested port, release artifact, or compatibility guarantee.

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
