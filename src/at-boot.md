# Starting services at boot

Enabling a unit creates a dependency symlink under a target.

```ini
[Install]
WantedBy=default.target
```

```sh
rystemctl --user enable hello.service
rystemctl --user is-enabled hello.service
rystemctl --user disable hello.service
```

`WantedBy=` creates a `.wants` symlink. `RequiredBy=` creates a `.requires`
symlink. `Alias=` and `Also=` are also supported.

## Targets

A target groups units. `default.target` is started when the manager starts.
Other common targets include `multi-user.target`, `timers.target`, and
`sockets.target`.

A custom target can group an application:

```sh
rystemctl --user start my-app.target
rystemctl --user stop my-app.target
```

## Dependencies and ordering

```ini
[Unit]
Description=Web application
Requires=postgres.service
After=postgres.service

[Service]
ExecStart=/usr/local/bin/webapp
```

- `Requires=` starts a required unit and propagates failure.
- `Wants=` starts an optional unit.
- `After=` and `Before=` set ordering without adding a dependency.
- `Conflicts=` stops the named unit when the current unit starts.

## Specifiers

Common specifiers include `%n` for unit name, `%p` for prefix, `%i` for instance,
`%u` for user, `%g` for group, `%h` for home, and `%t` for runtime directory.

Template units use `@`. Starting `web@1.service` loads `web@.service` and expands
`%i` to `1`.
