# Sockets & on-demand activation

On-demand activation is the trick where a service isn't running at all — until
something actually needs it. The manager holds a listening socket (or watches a
path), and the moment there's real traffic, it starts the service to handle it.
Services stay idle-forever when nobody comes calling.

This is how the pieces like journald and udev stay "off" until used, and it's a
handy pattern for your own "only when asked" daemons.

## Socket units

A `.socket` unit pairs with a service of the same base name. `my-api.socket`:

```ini
[Socket]
ListenStream=127.0.0.1:8080

[Install]
WantedBy=sockets.target
```

And `my-api.service` (same base name):

```ini
[Service]
Type=simple
ExecStart=/usr/bin/my-api
```

```sh
rystemctl --user start my-api.socket
```

The manager binds `127.0.0.1:8080` right away. The *process* only starts when a
connection arrives — so `my-api` is running exactly when it's being used, and
nothing is listening-but-orphaned otherwise.

### What it can listen on

| Directive | Listens on |
| --- | --- |
| `ListenStream=` | TCP or Unix-domain socket |
| `ListenDatagram=` | UDP or Unix datagram socket |
| `ListenSequentialPacket=` | a sequential-packet Unix socket |
| `ListenNetlink=` | a netlink socket (the socket used by e.g. udev) |

Unix-domain listeners (`/run/...`) are great for local IPC; TCP listeners work
for anything network-reachable.

## Path, mount, and device units

A **`.path` unit** starts a service when something *happens to the filesystem*:

```ini
[Path]
PathChanged=/var/spool/incoming

[Service]
ExecStart=/usr/bin/process-incoming
```

The trigger kinds are `PathExists=`, `PathExistsGlob=`,
`PathChanged=`, and `DirectoryNotEmpty=` — plus `MakeDirectory=` if the watched
directory should be created. This is the "a file landed, process it" pattern.

A **`.mount` unit** mounts a filesystem on `start` and unmounts it on `stop`
(from `[Mount]` `What=`/`Where=`/`Type=`/`Options=`). And **`.device` units** are
created automatically from the kernel's device tree, so a service can order
*after* a device (`After=dev-disk-by\x2duuid-...device`) and wait for hardware
before starting.

## Why you'd reach for this

- Services hold resources *only while in use* — nothing lingers waiting.
- The listening piece is always ready, so there's no "it's not up yet" window.
- Ordering: a unit that `After=`s a device or mount waits for real hardware /
  filesystems before it runs.

Next: [Starting services at boot](at-boot.md) — making units and their
sockets/timers start automatically.