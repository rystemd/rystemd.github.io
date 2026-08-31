# Installing rystemd

Every path below installs the three binaries — `rystemd` (the manager),
`rystemctl` (the `systemctl`-compatible CLI), and `rystemd-tui` (the terminal
client). Pick the one that matches your platform.

Installers are published as GitHub **release assets**; a package manager's job
is to fetch and place those. One-command installs below.

## Linux

### Fedora / RHEL / CentOS (rpm)

Grab the rpm for your architecture (`x86_64` or `aarch64`) from the
[release page](https://github.com/rystemd/rystemd/releases) and install it:

```sh
sudo dnf install ./rystemd-<ver>-1.x86_64.rpm    # or -aarch64.rpm
```

This places all three binaries in `/usr/bin` and registers them with rpm/dnf
for upgrade and removal.

### Debian / Ubuntu / Mint (deb)

Install the matching `.deb` from the
[release page](https://github.com/rystemd/rystemd/releases):

```sh
sudo apt install ./rystemd-<ver>-amd64.deb       # or -arm64.deb
```

Binaries land in `/usr/bin`; shell completions are installed to the standard
bash/zsh/fish completion directories.

> The `.deb`/`.rpm` are built straight from the same `scripts/package-linux.sh`
> that CI runs on every tag, so they are source-identical to the release.

### Homebrew (also good for immutable systems)

On Linux — including immutable Bazzite/Fedora-Atomic/Silverblue — install from
the rystemd tap without layering via `rpm-ostree`:

```sh
brew tap rystemd/rystemd
brew install rystemd
```

On an immutable image, Homebrew installs into its own prefix
(`/home/linuxbrew/.linuxbrew`) rather than touching the read-only `/usr`, so no
`rpm-ostree` layer or reboot is needed. The formula installs all three binaries
plus shell completions. It deliberately does **not** create a `systemctl`
symlink — on a systemd host that would shadow the real `systemctl` on your
PATH.

> Other package managers on an ostree host will try to write into the
> read-only system image. Prefer Homebrew there.

### Portable tarball (any distro)

Extract `rystemd-<ver>-<triple>.tar.gz` and add `bin/` to your `PATH`:

```sh
tar -xzf rystemd-<ver>-x86_64-unknown-linux-gnu.tar.gz
export PATH="$PWD/rystemd-<ver>-x86_64-unknown-linux-gnu/bin:$PATH"
```

The tarball's `bin/` contains the three executables, a `systemctl` symlink to
`rystemctl`, and the shell completion scripts. Use the `aarch64` tarball on
ARM64 hosts.

### From source (any distro)

Build from the [source repo](https://github.com/rystemd/rystemd):

```sh
cargo build --release
# binaries in target/release/: rystemd, rystemctl, rystemd-tui
```

## Windows

### Scoop (recommended)

The portable build is packaged as a [Scoop](https://scoop.sh) bucket
([`rystemd/scoop-rystemd`](https://github.com/rystemd/scoop-rystemd)):

```powershell
scoop bucket add rystemd https://github.com/rystemd/scoop-rystemd
scoop install rystemd/rystemd     # places rystemd, rystemctl, rystemd-tui on PATH
```

`checkver`/`autoupdate` are wired to the GitHub release stream, so `scoop
update` pulls new versions automatically on tag pushes.

### MSI (machine-wide)

Install `rystemd-<ver>-x86_64.msi` (an executable installer or
`msiexec /i rystemd-<ver>-x86_64.msi`). It places all three executables under
`%ProgramFiles%\rystemd` and registers with the Windows Installer for
upgrade/removal.

### NuGet (automation / private feeds)

The release ships `rystemd.<ver>.nupkg` as a **native tools package**:

```powershell
nuget install rystemd -OutputDirectory ./rystemd-pkg
# exes land under ./rystemd-pkg/rystemd.<ver>/tools/*
```

It ships the executables but does not shim `PATH` or register a service. Prefer
Scoop for an end-user PATH-installed setup, or the MSI for machine-wide.

### Portable zip

Extract `rystemd-<ver>-x86_64-pc-windows-msvc.zip` to a folder you add to
`PATH`. This is the same archive Scoop and the MSI are built from.

## Next

Once installed, open the manager for your own user (no root required) and poke
it — see [Getting started](getting-started.md).