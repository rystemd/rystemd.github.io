# Driving it: CLI, TUI, and the library

Three ways to talk to a running manager, from most everyday to most programmatic.

## The CLI (`rystemctl`)

`rystemctl` mirrors what you'd reach for from `systemctl`:

```sh
rystemctl --user start hello
rystemctl --user stop hello
rystemctl --user restart hello
rystemctl --user status hello
rystemctl --user is-active hello
rystemctl --user is-failed hello
rystemctl --user daemon-reload
rystemctl --user list-units
rystemctl --user list-timers
rystemctl --user journal hello       # the unit's captured output
```

On Linux, symlink it as `systemctl` and drop-in scripts work unchanged:

```sh
ln -s "$(command -v rystemctl)" ~/.local/bin/systemctl
```

> The manager and CLI are separate processes talking over a local socket, so
> there's no shell or D-Bus dependency between them. (A D-Bus surface exists
> too, and is what tooling that expects `org.freedesktop.systemd1` can talk to.)

**Shell completions** are built in:

```sh
rystemctl completions bash
rystemctl completions fish
rystemctl completions zsh
rystemctl completions powershell
rystemctl completions nushell
```

## The TUI (`rystemd-tui`)

A live, tabbed terminal view of the manager — **Units**, **Services**,
**Timers**, and **Unit files**, each with a status pane:

```sh
rystemd-tui --user
```

It connects to an already-running manager (it never starts a second one) and
detects the socket on its own. Keys are one-letter and obvious:

| Key | Action |
| --- | --- |
| `Tab` | switch tab |
| `↑`/`↓`/`j`/`k` | move |
| `/` | filter |
| `s` `x` `r` | start / stop / restart |
| `e` `d` | enable / disable |
| `R` | daemon-reload |
| `f` | refresh |
| `q` / `Esc` | quit |

## The library

If you're writing Rust and want to drive the manager from code, the same
interface the CLI uses is public:

```rust
use rystemd::control::{Control, SocketClient};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // true = user manager, false = system manager.
    let mut ctl = SocketClient::for_mode(true)?;

    ctl.start(&["hello.service"])?;
    ctl.restart(&["hello.service"])?;

    for s in ctl.status(&["hello.service"])? {
        println!("{}: {}/{}", s.name, s.active, s.sub);
    }

    ctl.stop(&["hello.service"])?;
    Ok(())
}
```

The same `Control` trait is implemented both by the in-process `Manager` and by
`SocketClient`, so your code can target either — swap the backend without
changing the call sites.

Next: go further upstream and manage the machine itself —
[Booting the machine itself](pid1.md).