# Introduction

**rystemd** is a small reimplementation of *what systemd does* — written in Rust
and built around three pieces you already know how to use:

- **`rystemd`** — the unit manager. It reads the same `.service`, `.timer`,
  `.socket`, `.target` files you already write and runs them the way systemd
  would.
- **`rystemctl`** — a `systemctl`-compatible command line. `start`, `stop`,
  `status`, `enable`, `list-timers`, and the rest work like you'd expect
  (and it can be symlinked as `systemctl` if you like).
- **`rystemd-tui`** — a live, tabbed terminal view of a running manager.

If you've written a systemd unit file, you already know most of rystemd. If you
haven't, that's fine too — this book is written for everyone from Sunday
tinkerers to folks who've managed init systems for decades.

## Why would you use it?

- **You want an init/unit manager you can read end-to-end.** rystemd is a few
  thousand lines you can actually audit.
- **You want a small footprint and no hidden machinery.** No async runtime, no
  surprise dependencies — what's compiled in is what you asked for.
- **You're curious how the pieces fit.** Because it mirrors systemd's managed
  unit files and lifecycle, it's a great way to *see* the machinery that's
  usually taken for granted.

## Where to go from here

This book is written so you can dive straight to whatever concerns you:

- Just want it running? → [Getting started](getting-started.md)
- Writing units? → [Writing your first service](services.md)
- Scheduling jobs? → [Timers: run things on a schedule](timers.md)
- Starting things automatically? → [Starting services at boot](at-boot.md)
- Using it as the machine's init? → [Booting the machine itself](pid1.md)
- On Windows? → [Platforms: Linux & Windows](platforms.md)

Linux is the primary, fully-featured target. Windows runs a compatibility port
of the same stack. See [Compatibility & known issues](compatibility.md) for the
honest state of both.