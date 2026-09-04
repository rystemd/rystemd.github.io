# Installing

Releases contain `rystemd`, `rystemctl`, and `rystemd-tui`. Download only from
the [GitHub release page](https://github.com/rystemd/rystemd/releases) or a
package definition that points to those release assets.

## Fedora, RHEL, and CentOS

Download the RPM matching the architecture, then install it:

```sh
sudo dnf install ./rystemd-<version>-1.x86_64.rpm
```

Use the `aarch64` RPM on ARM64. The package installs the three binaries under
`/usr/bin`.

On rpm-ostree systems, layer the same RPM into a new deployment:

```sh
rpm-ostree install ./rystemd-<version>-1.x86_64.rpm
rpm-ostree status
```

The currently booted deployment is unchanged. A reboot selects the new
deployment.

## Debian and Ubuntu

```sh
sudo apt install ./rystemd-<version>-amd64.deb
```

Use the `arm64` package on ARM64.

## Alpine Linux

The APK targets Alpine and musl.

```sh
apk add --no-cache --allow-untrusted --force-non-repository \
  https://github.com/rystemd/rystemd/releases/latest/download/rystemd-x86_64.apk
```

Use `rystemd-aarch64.apk` on ARM64. Current APKs are unsigned. The package
includes a `systemctl` symlink. It does not replace OpenRC or change the boot
configuration.

## Homebrew

Homebrew is the simplest installation method on immutable Fedora variants when
PID 1 replacement is not required.

```sh
brew tap rystemd/rystemd
brew install rystemd
```

Homebrew installs under its own prefix. It does not create a `systemctl`
symlink and does not configure rystemd as PID 1.

## Portable archives

Linux:

```sh
tar -xzf rystemd-<version>-x86_64-unknown-linux-gnu.tar.gz
export PATH="$PWD/rystemd-<version>-x86_64-unknown-linux-gnu/bin:$PATH"
```

The Linux archive includes `systemctl -> rystemctl`. Use the `aarch64` archive
on ARM64.

Windows releases include a portable zip, MSI, Scoop package, and NuGet package.

Scoop:

```powershell
scoop bucket add rystemd https://github.com/rystemd/scoop-rystemd
scoop install rystemd/rystemd
```

NuGet:

```powershell
nuget install rystemd -OutputDirectory ./rystemd-pkg
```

NuGet does not update `PATH` or register a service.

## Source builds

Source build and validation commands are maintained in the repository
[`DEV.md`](https://github.com/rystemd/rystemd/blob/main/DEV.md).

Installation does not replace the machine init. See
[Running as PID 1](pid1.md) for the separate boot procedure.
