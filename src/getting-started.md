# Getting started

Install the binaries first. See [Installing](install.md).

## Start a user manager

```sh
rystemd daemon --user
```

Keep that process running. Use another terminal for client commands:

```sh
rystemctl --user status
rystemctl --user list-units
```

The manager and client communicate through a local control socket.

## Test a service

Create `~/.config/systemd/user/hello.service`:

```ini
[Unit]
Description=Hello service

[Service]
Type=simple
ExecStart=/usr/bin/sleep 3600
Restart=on-failure
```

Load and control it:

```sh
rystemctl --user daemon-reload
rystemctl --user start hello.service
rystemctl --user status hello.service
rystemctl --user stop hello.service
```

Unit syntax is covered in [Writing a service](services.md).
