# Writing your first service

Units are plain text files. A unit is just: *what this is*, *what to run*, and
(usually) *when it should run*. The examples below use `rystemctl`; on Linux you
can symlink `rystemctl` as `systemctl` and the same commands work under that
name.

**Where files go (user mode):** `~/.config/systemd/user/`

**Where files go (system mode):** `/etc/systemd/system/`

If you're following along with `--user`, make sure `rystemd daemon --user` is
running first.

## 1. A basic service

```ini
[Unit]
Description=Hello world service

[Service]
Type=simple
ExecStart=/usr/bin/env sh -c 'while true; do echo hello; sleep 5; done'
Restart=on-failure
```

```sh
rystemctl --user start hello
rystemctl --user status hello
rystemctl --user stop hello
```

`Type=simple` means "go active as soon as the process is spawned". `Restart=`
decides what happens if it exits — more on that below.

## 2. A one-shot "state" service

Some units don't run forever — they do a job and finish. A common pattern is
`Type=oneshot` with `RemainAfterExit=yes`, which says: *the fact that this ran*
is the state, not a running process.

```ini
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/touch /tmp/rystemd-flag
ExecStop=/usr/bin/rm -f /tmp/rystemd-flag
```

```sh
rystemctl --user start flag.service
rystemctl --user is-active flag.service   # -> active, even though nothing runs
rystemctl --user stop flag.service        # runs ExecStop
```

Without `RemainAfterExit=yes`, a oneshot unit goes back to `inactive(dead)`
the moment it finishes — same as systemd.

## 3. A forking daemon (PIDFile)

Programs that daemonize themselves — the classic "fork into the background" —
need the manager told where the surviving process is:

```ini
[Service]
Type=forking
ExecStart=/usr/sbin/mydaemon
PIDFile=/run/mydaemon.pid
```

The manager waits for the parent to exit, then reads `PIDFile` to find the
process it should keep track of.

## 4. Writing units that restart

`Restart=` controls what happens when a service exits:

- `no` (default) — never restart.
- `on-failure` — restart unless it exited cleanly. The everyday choice.
- `always` — restart no matter what.
- `on-success`, `on-abnormal`, `on-abort`, `on-watchdog` — narrower variants.

Pair it with `RestartSec=` to pause between attempts (default is 100ms):

```ini
[Service]
Restart=on-failure
RestartSec=2s
```

A rate limit protects against tight crash/restart loops.

## 5. Controlling it without touching the file

You don't have to guess the whole unit up front. Environment, one-off args,
and tweaks can be layered on from the command line or with a `.d` override.

Layering from a drop-in — put this in
`~/.config/systemd/user/hello.service.d/override.conf`:

```ini
[Service]
Environment=GREETING=howdy
```

It merges over the base unit, so the stock file stays untouched and
re-updatable.

## 6. A few things people reach for

**Hold output and logs:** set `StandardOutput=` / `StandardError=` to
`journal` (the default), `inherit`, or `null` — or point them at a file with
`file:/path`.

**Limit resources:** `Nice=`, `UMask=`, `LimitNOFILE=`, `MemoryMax=`,
`TasksMax=`, `WorkingDirectory=`, `Environment=`, `EnvironmentFile=` all work
as you'd hope.

**Run as a different user:** `User=` / `Group=` (root manager only).

## Hardening, without the fine print

A large set of sandbox directives — `PrivateTmp=`, `PrivateDevices=`,
`ProtectSystem=`, `ProtectHome=`, `ReadOnlyPaths=`, `NoNewPrivileges=`,
`CapabilityBoundingSet=`, `SystemCallFilter=`, and more — are parsed and
honored. The honest caveat is in
[Compatibility & known issues](compatibility.md): a handful of hardening
directives are parsed-but-not-yet-enforced, and load with a warning so they
never silently *fail* your service.

## Filesystem layout, in one glance

| | system | user |
| --- | --- | --- |
| unit files | `/etc/systemd/system` | `~/.config/systemd/user` |
| enable/disable symlinks | `/etc/systemd/system` | `~/.config/systemd/user` |
| runtime data | `/run` | `$XDG_RUNTIME_DIR` |

Search order is highest-precedence first: `/etc` → `/run` → `/usr/lib` (and the
user equivalents).

Next: wire one up to run on its own — [Timers](timers.md) or
[Starting services at boot](at-boot.md).