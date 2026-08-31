# Booting the machine itself

Everything up to here has rystemd running *alongside* your normal init, driving
your own services and timers. There's a further step: **running rystemd as the
machine's init (PID 1)** — replacing systemd itself. That's possible, genuinely,
but it's an *advanced, experimental* thing, and this page is honest about what
it does and doesn't yet do so you don't end up with an unbootable machine.

## What "as PID 1" means

When rystemd is PID 1, the kernel hands it the whole machine. It has to:

- mount the API/virtual filesystems (`/proc`, `/sys`, `/dev`, `/run`, `/tmp`),
- run early-boot configuration (hostname, machine-id, sysctl, /etc/fstab),
- boot the default target and supervise everything,
- and power the machine off cleanly when told to.

That's built and works. What's less certain is the *very first* step of a real
boot — getting from the kernel's initramfs to the actual disk — and what to do
with a graphical desktop at the end of it.

## The honest state

| What you'd want | Status |
| --- | --- |
| Run as PID 1 inside an initramfs | ✅ works |
| Early-boot config + boot default target | ✅ works |
| **Switch out of the initramfs into the real disk (`/sysroot`)** | ✅ **now implemented** |
| Power off / reboot as PID 1 | ✅ works |
| Boot a graphical desktop (GNOME etc.) | ❌ not yet — boots to a text console |
| SELinux enforcing on a real Fedora host | ⚠️ policy module ships; needs live iteration |

So today: rystemd will happily boot a machine to a text console as init. A
current *desktop* install is still out of reach (no display-manager handoff
yet), and a real SELinux-enforcing Fedora host needs on-machine tuning. This is
a **test-in-a-VM-first** situation, not a firewall-and-click-it.

## Try it safely: the VM/test harness

The repo ships scripts (in `scripts/`, after cloning
[rystemd/rystemd](https://github.com/rystemd/rystemd)) that boot rystemd as
PID 1 for real, in qemu, with a busybox root — no disk, no risk:

```sh
# boot rystemd as PID 1 and drop into a serial console you can drive by hand
scripts/live-vm.sh

# automated: boot, run the boot-test unit, assert, power off
scripts/vm-test.sh

# exercise the /sysroot handoff end-to-end (boot a fake deployment, then
# switch into it and boot a unit that only exists there)
scripts/test-handoff.sh
```

Inside that environment you can run the same commands as anywhere:
`rystemctl list-units`, `rystemctl start demo.service`, `rystemd-tui`, and so
on — against the *real PID-1 manager*.

## On an immutable OS (Silverblue / Bazzite)

The goal of running rystemd as *the* init on an ostree system is the most
edited of all — those systems depend on systemd for ostree deployments,
updates, and the desktop session. The mechanism to pivot from the initramfs
into an ostree deployment is now implemented, but replacing systemd wholesale on
a machine you rely on isn't a supported setup yet. If you want to explore it,
keep it to a throwaway VM, and keep the stock systemd recoverable (you can
usually get back to it from the bootloader, but that's not a fun place to
discover things).

## How the pieces work (development detail)

The nitty-gritty — the switch_root sequence, initramfs detection, SELinux
policy, and the boot feature's implementation — lives in
[Development](development.md). For here, the short version is: rystemd detects
it's inside an initramfs, sees a real root staged at `/sysroot`, pivots into it,
and continues as init against the real disk.

Next: [Platforms: Linux & Windows](platforms.md), or
[Compatibility & known issues](compatibility.md) for the full picture.