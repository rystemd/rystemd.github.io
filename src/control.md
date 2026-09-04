# Control interfaces

## Command line

```sh
rystemctl --user start hello.service
rystemctl --user stop hello.service
rystemctl --user restart hello.service
rystemctl --user status hello.service
rystemctl --user is-active hello.service
rystemctl --user is-failed hello.service
rystemctl --user daemon-reload
rystemctl --user list-units
rystemctl --user list-timers
rystemctl --user journal hello.service
```

The manager and client communicate through a local socket. Linux builds may
also expose a partial `org.freedesktop.systemd1` D-Bus surface.

Shell completions:

```sh
rystemctl completions bash
rystemctl completions fish
rystemctl completions zsh
rystemctl completions powershell
rystemctl completions nushell
```

## Terminal interface

```sh
rystemd-tui --user
```

The TUI connects to an existing manager.

| Key | Action |
| --- | --- |
| `Tab` | Change tab |
| `Up`, `Down`, `j`, `k` | Move selection |
| `/` | Filter |
| `s`, `x`, `r` | Start, stop, restart |
| `e`, `d` | Enable, disable |
| `R` | Reload units |
| `f` | Refresh |
| `q`, `Esc` | Quit |

## Rust API

```rust
use rystemd::control::{Control, SocketClient};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut ctl = SocketClient::for_mode(true)?;
    ctl.start(&["hello.service"])?;

    for status in ctl.status(&["hello.service"])? {
        println!("{}: {}/{}", status.name, status.active, status.sub);
    }

    ctl.stop(&["hello.service"])?;
    Ok(())
}
```

`Manager` and `SocketClient` implement the same `Control` trait.
