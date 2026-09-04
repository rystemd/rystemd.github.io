# Writing a service

User units live in `~/.config/systemd/user/`. System units live in
`/etc/systemd/system/`.

## Long-running service

```ini
[Unit]
Description=Example worker

[Service]
Type=simple
ExecStart=/usr/bin/env sh -c 'while true; do echo hello; sleep 5; done'
Restart=on-failure
RestartSec=2s
```

```sh
rystemctl --user start hello.service
rystemctl --user status hello.service
rystemctl --user stop hello.service
```

`Type=simple` becomes active after process creation. `Restart=on-failure`
restarts the service after an unsuccessful exit.

## Oneshot service

```ini
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/touch /tmp/rystemd-flag
ExecStop=/usr/bin/rm -f /tmp/rystemd-flag
```

`RemainAfterExit=yes` keeps the unit active after `ExecStart` exits. Stopping the
unit runs `ExecStop`.

## Forking service

```ini
[Service]
Type=forking
ExecStart=/usr/sbin/mydaemon
PIDFile=/run/mydaemon.pid
```

The manager waits for the parent to exit, then tracks the process named by
`PIDFile=`.

## Restart policy

Common values:

| Value | Behavior |
| --- | --- |
| `no` | Never restart |
| `on-failure` | Restart after an unsuccessful exit |
| `always` | Restart after every exit |
| `on-success` | Restart after a successful exit |
| `on-abnormal` | Restart after abnormal termination |

`RestartSec=` controls the delay between attempts. A rate limit prevents tight
restart loops.

## Drop-ins

A drop-in changes a unit without editing the base file. Example path:

```text
~/.config/systemd/user/hello.service.d/override.conf
```

```ini
[Service]
Environment=GREETING=howdy
```

Run `rystemctl --user daemon-reload` after changing unit files.

## Common service settings

- Account: `User=`, `Group=`
- Paths: `WorkingDirectory=`, `EnvironmentFile=`
- Output: `StandardOutput=`, `StandardError=`
- Limits: `MemoryMax=`, `TasksMax=`, `LimitNOFILE=`
- Sandbox: `PrivateTmp=`, `ProtectSystem=`, `ProtectHome=`,
  `NoNewPrivileges=`, `CapabilityBoundingSet=`, `SystemCallFilter=`

Some directives remain partial. See [Compatibility](compatibility.md).
