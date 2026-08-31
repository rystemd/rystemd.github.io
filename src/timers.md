# Timers: run things on a schedule

A timer is a small unit that wakes up a matching service on a schedule. The
service does the work; the timer decides when. Like systemd — if you know
`systemd.timer`, this is the same idea.

## A daily timer

`~/.config/systemd/user/daily-backup.timer`:

```ini
[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=yes

[Install]
WantedBy=timers.target
```

And the work it triggers — `daily-backup.service`, same base name so the timer
finds it automatically:

```ini
[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

```sh
rystemctl --user start daily-backup.timer
rystemctl --user list-timers       # see next fire time
rystemctl --user status daily-backup.timer
```

`WantedBy=timers.target` under `[Install]` means "start me as a regular timer",
so all your timers get grouped under one target.

## Calendar schedules you can read

`OnCalendar=` understands the full systemd calendar grammar. A few to get you
started:

| Schedule | Meaning |
| --- | --- |
| `daily` | once a day (midnight) |
| `weekly` | once a week (Mon 00:00) |
| `Mon..Fri 09:00` | weekdays at 9am |
| `*:0/15` | every 15 minutes |
| `2026-08-21 09:00` | a one-shot date |
| `Mon,Wed 09:00/2` | Monday & Wednesday, 9:00 and 9:02 |

`Persistent=yes` is worth knowing: if the machine was off at the scheduled time,
it runs soon after it comes back, rather than skipping the missed run.

## Timers that aren't calendar-based

Not everything is "at a time of day." `OnBootSec=5min` runs five minutes after
boot; `OnUnitActiveSec=1h` runs an hour after the *unit* last ran; combined.
`OnBootSec` + `OnUnitActiveSec` is the classic "retry loop" recipe — a watchdog
that re-runs some interval after its previous attempt.

```ini
[Timer]
OnBootSec=5min
OnUnitActiveSec=10min   # every 10 min after the last fire
```

## Did it run?

```sh
rystemctl --user list-timers                 # next elapse per timer
rystemctl --user status daily-backup.service # last run + captured output
```

Next: make services (not just timers) start automatically —
[Starting services at boot](at-boot.md).