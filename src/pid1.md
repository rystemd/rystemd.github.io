# Running as PID 1

PID 1 support is experimental. Console boot works. A stock Fedora graphical
session does not.

Current limits:

- SELinux policy requires live validation. Use permissive mode for testing.
- The stock Fedora `default.target` graph is not supported as a whole.
- Graphical login and desktop session startup are incomplete.
- The stock initramfs must mount and stage an ostree deployment.

Start with the VM scripts in the source repository:

```sh
scripts/live-vm.sh
scripts/vm-test.sh
scripts/test-handoff.sh
```

The host procedure below is for an rpm-ostree VM or disposable installation.
It keeps the previous deployment bootable with systemd.

## Reversible rpm-ostree procedure

The rollback condition determines the layout:

- The stock `/sbin/init` link remains unchanged.
- `/usr/lib/systemd/systemd` remains installed.
- `/usr/bin/systemctl` remains unchanged.
- rystemd uses a separate wrapper and private configuration directory.
- The previous ostree deployment retains its original kernel arguments.

Recovery then requires only a reboot into the previous boot entry.

### 1. Record the current deployments

```sh
rpm-ostree status
ostree admin status
```

Confirm that the bootloader lists the current and previous deployments. Console
or out-of-band bootloader access is required.

### 2. Install rystemd

RPM overlay:

```sh
rpm-ostree install ./rystemd-<version>-1.x86_64.rpm
rpm-ostree status
```

Use the matching architecture. The command creates a new deployment. The
running deployment and previous boot entry remain available.

Homebrew can also supply the binaries:

```sh
brew tap rystemd/rystemd
brew install rystemd
command -v rystemd
command -v rystemctl
```

Homebrew paths depend on the installation prefix. Both binaries must be visible
after the ostree deployment is staged. Record their absolute paths for the next
steps.

### 3. Create private boot configuration

```sh
sudo install -d -m 0755 /etc/rystemd/system /etc/rystemd/bin
sudo ln -sfn /usr/lib/systemd/system/multi-user.target   /etc/rystemd/system/default.target
sudo ln -sfn /usr/bin/rystemctl /etc/rystemd/bin/systemctl
```

Substitute the Homebrew `rystemctl` path when using Homebrew.

The private `default.target` selects console and multi-user boot for rystemd.
The stock `/etc/systemd/system/default.target` is not changed. The private
`systemctl` link is available only through the wrapper PATH. Units containing
an absolute `/usr/bin/systemctl` still call the stock client and require
separate testing.

### 4. Create the init wrapper

RPM installation:

```sh
sudo tee /etc/rystemd/init >/dev/null <<'EOF'
#!/bin/sh
export RYSTEMD_CONFIG_DIR=/etc/rystemd/system
export RYSTEMD_UNIT_PATH=/etc/rystemd/system:/etc/systemd/system:/run/systemd/system:/usr/lib/systemd/system
export PATH=/etc/rystemd/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
exec /usr/bin/rystemd daemon "$@"
EOF
sudo chmod 0755 /etc/rystemd/init
```

Substitute the absolute Homebrew `rystemd` path on the final line when using
Homebrew.

Verify all paths before rebooting:

```sh
test -x /etc/rystemd/init
test -e /etc/rystemd/system/default.target
test -x /usr/bin/rystemd
```

Use the recorded Homebrew path instead of `/usr/bin/rystemd` when applicable.

### 5. Add a separate boot configuration

Keep the stock initramfs. It handles disk discovery, encryption, ostree
selection, `/sysroot`, `/var`, and `/etc` setup.

Append these kernel arguments only to the experimental entry:

```text
init=/etc/rystemd/init enforcing=0
```

`init=` selects the post-initramfs PID 1. Do not replace `rdinit=/init` in the
stock initramfs. Do not add `systemd.unit=`; rystemd selects the private
`default.target` created above.

For the first test, edit a boot entry once from the bootloader. For a persistent
entry, use deployment-specific rpm-ostree kernel arguments:

```sh
rpm-ostree kargs --append=init=/etc/rystemd/init --append=enforcing=0
rpm-ostree status
```

Run this against the new rystemd deployment. Do not alter or remove the previous
deployment. Bootloader details differ between GRUB and systemd-boot, but both
must retain the previous deployment entry without the rystemd `init=` argument.

### 6. Verify the experimental boot

```sh
cat /proc/cmdline
cat /proc/1/comm
rystemctl list-units
rystemctl status default.target
```

Expected values:

- `/proc/cmdline` contains `init=/etc/rystemd/init` and `enforcing=0`.
- `/proc/1/comm` reports `rystemd`.
- `default.target` is active.
- A console login is available.

Do not enable graphical services until their dependencies have been tested.

## Reversal

1. Reboot or reset the machine.
2. Select the previous ostree deployment in the bootloader.
3. Confirm that `/proc/1/comm` reports `systemd`.

No symlink repair, initramfs rebuild, package removal, or kernel argument edit is
required for recovery. Cleanup can wait until after the previous deployment has
booted successfully.

Global replacement of `/sbin/init`, `/usr/bin/systemctl`, or the stock
initramfs is outside this procedure because it breaks reboot-only recovery.
