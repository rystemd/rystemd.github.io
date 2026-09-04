# Timers

A timer starts a service on a schedule. Matching names pair automatically. For
example, `daily-backup.timer` starts `daily-backup.service`.

`~/.config/systemd/user/daily-backup.timer`:

```ini
[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=yes

[Install]
WantedBy=timers.target
```

`~/.config/systemd/user/daily-backup.service`:

```ini
[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

```sh
rystemctl --user start daily-backup.timer
rystemctl --user list-timers
rystemctl --user status daily-backup.timer
```

## Calendar values

| Value | Meaning |
| --- | --- |
| `daily` | Every day at midnight |
| `weekly` | Monday at midnight |
| `Mon..Fri 09:00` | Weekdays at 09:00 |
| `*:0/15` | Every 15 minutes |
| `2026-08-21 09:00` | One specific date and time |

`Persistent=yes` runs a missed calendar event after the manager starts again.

## Monotonic values

```ini
[Timer]
OnBootSec=5min
OnUnitActiveSec=10min
```

`OnBootSec=` is measured from boot. `OnUnitActiveSec=` is measured from the last
activation.
