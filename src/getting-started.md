# Getting started

The quickest path to a running manager and seeing it work.

## Install

Install `rystemd`, `rystemctl`, and `rystemd-tui` for your platform — see
[Installing](install.md).

## Start it, then poke it

Run a manager for your own user (no root, no system changes):

```sh
rystemd daemon --user
```

In another terminal, ask the manager what it knows:

```sh
rystemctl --user status
rystemctl --user list-units
```

The manager and CLI are separate processes that talk over a local socket, so
this "run the daemon, talk to it with the CLI" shape is how everything works.

## Confirm it's real: a 30-second service

Create `~/.config/systemd/user/hello.service`:

```ini
[Unit]
Description=Hello world

[Service]
Type=simple
ExecStart=/usr/bin/sleep 3600
Restart=on-failure
```

Tell the manager about it, start it, and look at it:

```sh
rystemctl --user daemon-reload
rystemctl --user start hello
rystemctl --user status hello
rystemctl --user stop hello
```

That's the whole loop. From here the natural next step is writing units that
fit what you actually want — head to [Writing your first service](services.md).