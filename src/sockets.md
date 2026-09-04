# Socket and path activation

A socket unit owns a listener and starts its matching service when traffic
arrives.

`my-api.socket`:

```ini
[Socket]
ListenStream=127.0.0.1:8080

[Install]
WantedBy=sockets.target
```

`my-api.service`:

```ini
[Service]
Type=simple
ExecStart=/usr/bin/my-api
```

```sh
rystemctl --user start my-api.socket
```

Supported listener directives include `ListenStream=`, `ListenDatagram=`,
`ListenSequentialPacket=`, and `ListenNetlink=`. Linux services receive socket
file descriptors through `LISTEN_FDS` and `LISTEN_PID`.

## Path activation

A path unit starts a service after a filesystem condition changes:

```ini
[Path]
PathChanged=/var/spool/incoming

[Install]
WantedBy=paths.target
```

Matching directives include `PathExists=`, `PathExistsGlob=`, `PathChanged=`,
and `DirectoryNotEmpty=`.

## Mount and device units

A mount unit uses `What=`, `Where=`, `Type=`, and `Options=`. Start mounts the
filesystem. Stop unmounts it.

Linux device units are created from the kernel device tree. Dependencies can
order a service after a device unit.
