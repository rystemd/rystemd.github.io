# Starting services at boot

Two ideas decide what runs when the machine (or your user session) comes up:
**enabling** a unit, and **targets**.

## Enable = "start me at boot"

`enable` wires a unit into the boot-up graph by creating a `.wants` symlink —
the same mechanism systemd uses. Give a unit an `[Install]` section declaring
*where* it wants to live, then enable it:

```ini
[Install]
WantedBy=default.target
```

```sh
rystemctl --user enable hello.service     # creates a .wants symlink
rystemctl --user is-enabled hello.service # -> enabled
rystemctl --user disable hello.service
```

`RequiredBy=` creates a `.requires` symlink instead (boot fails if the unit
doesn't come up); `Alias=` and `Also=` are honored too.

## Targets: "boot into this posture"

A `.target` unit is just a name for "a collection of things that should be up."
`default.target` is whatever your setup boots into; you'll also meet
`multi-user.target`, `timers.target`, and `sockets.target`. Enabling a unit
under a target's name pulls it in when that target starts — which is why
timers and sockets carry `WantedBy=timers.target` / `WantedBy=sockets.target`.

You can define your own targets to group related work, write a unit with
`WantedBy=my-app.target`, and drive the whole group with:

```sh
rystemctl --user start my-app.target
rystemctl --user stop my-app.target
```

## Dependencies & ordering

When things must start in a certain order or not at all without each other,
use `[Unit]` directives:

```ini
[Unit]
Description=Web app
Requires=postgres.service     # start this too; if it fails, we don't start
After=postgres.service       # ...but wait for it first

[Service]
ExecStart=/usr/local/bin/webapp
```

The three you'll reach for:

- `Requires=` — a hard dependency: it gets started too, and if it fails this
  unit doesn't start.
- `Wants=` — a soft dependency: started too, but failure is ignored.
- `After=` / `Before=` — *ordering only*: control start order without implying
  a dependency.

`Conflicts=` stops the named unit when this one starts. (`After=` without
`Requires=`/`Wants=` is a common gotcha — it orders but doesn't pull in.)

## Drop-ins & specifiers (tweaking without editing)

**Drop-ins.** Override a stock unit without touching the original — a file in
`<name>.service.d/`:

```ini
# ~/.config/systemd/user/hello.service.d/override.conf
[Service]
Environment=GREETING=howdy
```

**Specifiers.** Values that expand at load time: `%n` (unit name), `%p`
(prefix before `@`), `%i` (instance), `%u`/`%g` (user/group), `%h` (home),
`%t` (runtime dir), `%%` (a literal `%`). Instanced units — `web@1.service` —
use `%i` to vary per instance and are the tidy way to run several copies of one
unit.

Next: [Driving it: CLI, TUI, and the library](control.md) for the day-to-day
commands and live views.