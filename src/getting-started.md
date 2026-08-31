# Getting started

The quickest path to a running manager and seeing it work.

## Install

Pick whichever fits your machine. All of these give you the three binaries
`rystemd`, `rystemctl`, and `rystemd-tui`.

**Linux (Fedora / Bazzite / other immutable distros)** — via Homebrew:

```sh
brew tap rystemd/rystemd
brew install rystemd
```

> On an immutable host (Bazzite, Fedora Atomic, Silverblue) use brew rather
> than `rpm-ostree` — it installs into Homebrew's own prefix without touching
> the read-only system.

**Windows (Scoop)** — the tidy, PATH-installed choice:

```powershell
scoop bucket add rystemd https://github.com/rystemd/scoop-rystemd
scoop install rystemd/rystemd
```

**Windows (NuGet)** — for automation / private feeds:

```powershell
nuget install rystemd -OutputDirectory ./rystemd-pkg
```

**Windows (MSI)** — a machine-wide installer: grab
`rystemd-<ver>-x86_64.msi` from the [release page](https://github.com/rystemd/rystemd/releases).

**Anything (from source):**

```sh
cargo build --release
# binaries in target/release/: rystemd, rystemctl, rystemd-tui
```

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