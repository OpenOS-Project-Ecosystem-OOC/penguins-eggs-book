# Chapter 14 # Integrations ecosystem

The `all-features` branch integrates companion tools alongside penguins-eggs.
Each tool has bidirectional hooks with `eggs produce` and related commands, and
registers itself as an eggs plugin so its state is embedded into produced ISOs.

## Overview

| Tool | Language | Purpose |
|---|---|---|
| [penguins-recovery](#penguins-recovery) | Shell | Unified rescue toolkit |
| [penguins-powerwash](#penguins-powerwash) | Shell | Factory reset |
| [penguins-immutable-framework](#penguins-immutable-framework-pif) | Go + Shell | Immutable Linux framework |
| [penguins-kernel-manager](#penguins-kernel-manager-pkm) | Python | Kernel lifecycle management |
| [penguins-incus-hub](#penguins-incus-hub) | Shell | Incus guest management via penguins-incus-platform |

---

## penguins-recovery

[penguins-recovery](https://github.com/Interested-Deving-1896/penguins-recovery)
is a unified Linux system recovery toolkit. It layers recovery tools onto any
penguins-eggs naked ISO and supports six distro families.

### Supported distro families

| Family | Package manager | Distros |
|--------|----------------|---------|
| Debian | apt | Debian, Ubuntu, Pop!\_OS, Linux Mint, LMDE, Devuan, MX, Zorin, elementary |
| Fedora/RHEL | dnf/yum | Fedora, AlmaLinux, Rocky Linux, CentOS, Nobara |
| Arch | pacman | Arch, EndeavourOS, Manjaro, BigLinux, Garuda, CachyOS |
| SUSE | zypper | openSUSE Leap/Tumbleweed/Slowroll, SLES |
| Alpine | apk | Alpine Linux |
| Gentoo | emerge | Gentoo, Funtoo, Calculate |

### Using the adapter

The adapter system layers recovery tools onto a penguins-eggs naked ISO. It
auto-detects the distro family from `/etc/os-release`:

```
# Basic: layer recovery tools onto a naked ISO
sudo make adapt INPUT=naked-debian-bookworm-amd64.iso

# With custom output name
sudo make adapt INPUT=naked-arch-amd64.iso OUTPUT=recovery-arch.iso

# Include rescapp GUI wizard
sudo make adapt INPUT=naked-ubuntu-noble-amd64.iso RESCAPP=1

# With GUI profile (minimal, touch, or full)
sudo make adapt INPUT=naked-debian-bookworm-amd64.iso GUI=minimal
sudo make adapt INPUT=naked-ubuntu-noble-amd64.iso GUI=touch RESCAPP=1
sudo make adapt INPUT=naked-arch-amd64.iso GUI=full

# Direct script usage
sudo ./adapters/adapter.sh --input naked.iso --output recovery.iso --gui minimal
```

The `--recovery` flag on `eggs produce` calls this adapter automatically
(see Chapter 7).

### Builders

penguins-recovery includes several rescue image builders:

| Builder | Based on | Description |
|---------|----------|-------------|
| `debian/` | Debian mini-rescue | Debian-based rescue Live CD |
| `arch/` | platter-engineer | Arch-based disk rescue image |
| `uki/` | rescue-image1 | Unified Kernel Image rescue |
| `uki-lite/` | host kernel | Lightweight UKI via objcopy |
| `lifeboat/` | Alpine | Single-file UEFI EFI |
| `rescatux/` | live-build | Rescatux live-build rescue CD |

### rescapp

rescapp is a Qt5/kdialog GUI rescue wizard included in penguins-recovery. It
guides users through common recovery tasks: GRUB repair, password reset,
filesystem check, and chroot operations. Enable it with `--recovery-rescapp`
on `eggs produce`, or with `RESCAPP=1` on the adapter.

### Installation

```
git clone https://github.com/Interested-Deving-1896/penguins-recovery
cd penguins-recovery
sudo make install
```

---

## penguins-powerwash

[penguins-powerwash](https://github.com/Interested-Deving-1896/penguins-powerwash)
is a distro-agnostic, filesystem-agnostic factory reset tool for Linux.

### Reset modes

| Mode | What it does |
|------|-------------|
| `soft` | Removes user data, preserves system packages |
| `medium` | Removes user data and user-installed packages |
| `hard` | Full factory reset to a clean system state |
| `sysprep` | Prepares the system for imaging (removes machine-specific data) |
| `hardware` | Wipes all data including the bootloader |

### Usage

```
sudo penguins-powerwash soft
sudo penguins-powerwash medium
sudo penguins-powerwash hard
sudo penguins-powerwash sysprep --shutdown
sudo penguins-powerwash --dry-run hard
sudo penguins-powerwash backup create --encrypt
```

The `--dry-run` flag prints every action without executing it.

### Supported systems

Distros: Debian, Ubuntu, Fedora, RHEL, Arch, openSUSE, Gentoo, Void
(auto-detected). Filesystems: ext4, xfs, btrfs (native snapshots), ZFS
(native snapshots), overlayfs.

### Integration with penguins-eggs

penguins-powerwash has bidirectional integration with penguins-eggs:

| Event | Action |
|---|---|
| Pre-reset (any mode) | Calls `eggs produce --naked` to snapshot the live system state before wiping |
| Pre-reset (any mode) | Calls `penguins-recovery snapshot create pre-powerwash-<ts>` if recovery is present |
| Post-reset (hard/sysprep) | Calls `penguins-recovery adapter.sh` to re-layer recovery tools |
| Post-backup | Notifies eggs so the backup path is recorded in the next ISO manifest |

Configure in `/etc/penguins-powerwash/eggs-hooks.conf`:

```bash
EGGS_BIN="/usr/bin/eggs"
RECOVERY_BIN="/usr/bin/penguins-recovery"
PRE_RESET_SNAPSHOT=1
PRE_RESET_EGGS_PRODUCE=0  # set 1 to produce a naked ISO before reset
POST_HARD_RESET_ADAPT=1   # re-layer recovery tools after hard reset
```

penguins-powerwash also ships an eggs plugin that embeds the powerwash binary
and config into produced ISOs and adds a "Factory Reset" GRUB menu entry.

### Installation

```
git clone https://github.com/Interested-Deving-1896/penguins-powerwash
cd penguins-powerwash
sudo make install
```

---

## penguins-immutable-framework (PIF)

[penguins-immutable-framework](https://github.com/Interested-Deving-1896/penguins-immutable-framework)
is a distro-agnostic framework for building immutable Linux distributions. It
provides a unified CLI and HAL over multiple immutability backends.

### Backends

| Backend | Mechanism | Best for |
|---------|-----------|----------|
| `abroot` | A/B partition swap + OCI images | Appliance/desktop, atomic OCI-based updates |
| `ashos` | BTRFS snapshot tree | Multi-distro, hierarchical snapshot management |
| `frzr` | Read-only BTRFS subvolume deploy | Gaming/appliance, image-based deployment |
| `akshara` | YAML-declared system rebuild | Declarative, container-native distros |
| `btrfs-dwarfs` | BTRFS + DwarFS hybrid blend layer | Storage-constrained, high-compression roots |

### Quick start

```
# Install PIF
curl -fsSL https://pif.example.org/install.sh | sh

# Initialise with the abroot backend
pif init --distro ubuntu --backend abroot --arch x86_64

# Upgrade (backend-transparent)
pif upgrade

# Enter mutable mode for system changes
pif mutable enter
# ... make changes ...
pif mutable exit

# Roll back to previous state
pif rollback
```

### Integration with penguins-eggs

| Event | Action |
|---|---|
| `pif upgrade` (pre) | Calls `penguins-recovery snapshot create pre-pif-upgrade` |
| `pif upgrade` (post) | Notifies eggs via `eggs pif-upgraded` hook |
| `pif mutable enter` | Warns eggs that the system is temporarily mutable |
| `pif mutable exit` | Triggers `eggs produce --update-root` (if configured) |
| `pif rollback` | Calls `penguins-recovery snapshot create pre-pif-rollback` |

Configure in `pif.toml`:

```toml
[hooks]
eggs_bin = "/usr/bin/eggs"
recovery_bin = "/usr/bin/penguins-recovery"
pre_upgrade_snapshot = true
post_upgrade_notify = true
mutable_warn_eggs = true
post_mutable_produce = false  # set true to auto-produce ISO after mutable exit
pre_rollback_snapshot = true
```

---

## penguins-kernel-manager (PKM)

[penguins-kernel-manager](https://github.com/Interested-Deving-1896/penguins-kernel-manager)
manages the full Linux kernel lifecycle — fetch, patch, configure, compile,
package, install, hold, and remove — across all major distributions and CPU
architectures.

### Kernel sources

| Source | Architectures |
|--------|--------------|
| Ubuntu Mainline PPA | amd64, arm64, armhf, ppc64el, s390x, i386 |
| XanMod | amd64 (v1–v4, edge, lts, rt) |
| Liquorix | amd64 |
| Distro-native (system package manager) | all |
| Gentoo source compilation | all |
| Local file (.deb / .rpm / .pkg.tar.* / .apk / .xbps) | all |
| lkf build (compile from source) | all |

### Supported distributions

Any distro using one of: `apt`, `pacman`, `dnf`, `zypper`, `apk`, `portage`,
`xbps`, `nix`.

### Usage

```
# List available kernels
pkm list

# Install a specific kernel
pkm install 6.9.0 --source mainline

# Build from source using an lkf profile
pkm build --profile server-optimized

# Hold a kernel (prevent removal)
pkm hold 6.9.0

# Remove old kernels
pkm remove --old

# GUI mode
pkm gui
```

### Integration with penguins-eggs

| Event | Action |
|---|---|
| Pre-install | Calls `penguins-recovery snapshot create pre-kernel-<version>` |
| Post-install | Notifies eggs via `eggs kernel-changed` hook |
| Pre-remove | Warns if the kernel being removed is embedded in the last eggs ISO |
| Post-remove-old | Triggers `eggs produce --update-kernel-list` (non-blocking) |

Configure in `/etc/penguins-kernel-manager/hooks.conf`:

```toml
[hooks]
eggs_bin = "/usr/bin/eggs"
recovery_bin = "/usr/bin/penguins-recovery"
pre_install_snapshot = true
post_install_notify = true
pre_remove_warn = true
post_remove_old_sync = false  # set true to rebuild ISO automatically
```

### Installation

```
git clone https://github.com/Interested-Deving-1896/penguins-kernel-manager
cd penguins-kernel-manager
sudo make install
```

---

## penguins-incus-hub

[penguins-incus-hub](https://github.com/Interested-Deving-1896/penguins-eggs/tree/all-features/integrations/penguins-incus-hub)
is the integration layer connecting
[penguins-incus-platform](https://github.com/Interested-Deving-1896/penguins-incus-platform)
(PIP) with penguins-eggs.

PIP is a unified Incus container and VM management platform with three
frontends — a Qt6/QML desktop app, a React web UI, and a CLI — backed by a
single FastAPI + D-Bus daemon. Four previously independent toolkits are merged
into the daemon as provisioning plugins:

| Source project | Guest type | CLI entry point |
|---|---|---|
| incusbox | Generic Linux containers | `penguins-incus provision generic` |
| waydroid-toolkit | Waydroid (Android) containers | `penguins-incus provision waydroid` |
| Incus-MacOS-Toolkit | macOS KVM VMs | `penguins-incus provision macos` |
| incus-windows-toolkit | Windows VMs | `penguins-incus provision windows` |

### What penguins-incus-hub does

The hub embeds the PIP daemon and CLI into produced ISOs so that any
penguins-eggs live system can manage Incus guests without additional setup.

| Event | Action |
|---|---|
| `eggs produce` (post) | Copies `penguins-incus-daemon` binary into the ISO |
| `eggs produce` (post) | Copies the `penguins-incus` CLI into the ISO |
| `eggs produce` (post) | Copies 16 bundled Incus profile presets into the ISO |
| `eggs produce` (post) | Writes a systemd unit so the daemon auto-starts in the live environment |
| Pre-reset (any powerwash mode) | Snapshots all running Incus containers and VMs |
| Post-hard-reset / sysprep | Restarts the PIP daemon and re-applies default profiles |

### Hook configuration

`/etc/penguins-incus-hub/eggs-hooks.conf`:

```bash
# Path to the penguins-incus-platform installation
PIP_ROOT="/usr/lib/penguins-incus-platform"

# Embed the PIP daemon binary into produced ISOs (default: 1)
EMBED_DAEMON=1

# Embed the penguins-incus CLI into produced ISOs (default: 1)
EMBED_CLI=1

# Embed bundled Incus profiles into produced ISOs (default: 1)
EMBED_PROFILES=1

# Snapshot all running Incus instances before any powerwash reset (default: 1)
PRE_RESET_SNAPSHOT=1

# Restart daemon after hard or sysprep reset (default: 1)
POST_HARD_RESET_RESTART=1
```

### What gets embedded in the ISO

| Path in ISO | Content |
|---|---|
| `/usr/local/bin/penguins-incus-daemon` | PIP daemon binary |
| `/usr/local/bin/penguins-incus` | PIP CLI binary |
| `/usr/local/share/penguins-incus-platform/profiles/` | 16 bundled Incus profile presets |
| `/etc/systemd/system/penguins-incus-daemon.service` | Systemd unit (auto-start in live env) |

### Using Incus guest management in a live ISO

Once booted into a penguins-eggs live system with the hub embedded:

```
# List running containers and VMs
penguins-incus container list
penguins-incus vm list

# Create a generic Linux container
penguins-incus provision generic create mybox --image images:ubuntu/24.04/cloud

# Create a Waydroid (Android) container
penguins-incus provision waydroid create my-android --image-type GAPPS

# Create a macOS KVM VM
penguins-incus provision macos image firmware
penguins-incus provision macos image fetch --version sonoma
penguins-incus provision macos create my-mac --version sonoma

# Create a Windows VM
penguins-incus provision windows create my-win --image /path/to/win11.iso
```

The web UI is also available at `http://localhost:8765` once the daemon starts.

### Installation

```
# 1. Install penguins-incus-platform
git clone https://github.com/Interested-Deving-1896/penguins-incus-platform
cd penguins-incus-platform
pip install -e "penguins-incus-platform/daemon[dev]"
pip install -e "penguins-incus-platform/cli[dev]"

# 2. Register the eggs and recovery plugins
sudo penguins-incus-hub install
```

Or manually:

```
sudo ln -s /usr/share/penguins-incus-hub/integration/eggs-plugin/pip-hook.sh \
           /usr/share/penguins-eggs/plugins/pip-hook.sh

sudo ln -s /usr/share/penguins-incus-hub/integration/recovery-plugin/pip-recovery-plugin.sh \
           /usr/share/penguins-recovery/plugins/pip-recovery-plugin.sh
```

---

## Plugin system

Each ecosystem tool registers an eggs plugin and (where applicable) a recovery
plugin:

- `<tool>/integration/eggs-plugin/` — called by `eggs produce` to embed tool
  state and binaries into the ISO and coordinate lifecycle events
- `<tool>/integration/recovery-plugin/` — called by penguins-recovery before
  and after factory resets

The 46 lightweight plugins in `integrations/plugins/` extend eggs across six
domains:

| Domain | Count | Key projects |
|--------|-------|-------------|
| Build Infrastructure | 14 | dwarfs, erofs, verity, mkosi, buildroot, gpt-image, partymix |
| Distribution | 3 | git-lfs, opengist, gentoo-installer |
| Dev Workflow | 2 | ts-ci, security-scan (VerityOps) |
| Decentralized | 4 | brig, git-lfs-ipfs, ipgit, git-ipfs-rehost |
| Config Management | 7 | presslabs/gitfs, dsxack/gitfs, wardrobe merge tools |
| Packaging | 3 | gitpack, github-paser, github-directory-downloader |

See `integrations/INTEGRATIONS.md` and `integrations/INTEGRATION-SPEC.md` for
the full plugin list and per-plugin specifications.
