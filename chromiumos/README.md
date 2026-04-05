# ChromiumOS in Penguins-Eggs

Architecture-agnostic reference for all ChromiumOS-specific directories and
files in this repository. The goal is to make ChromiumOS development
reproducible across amd64, arm64, i386, and riscv64.

---

## Table of Contents

- [Overview](#overview)
- [Supported Derivatives](#supported-derivatives)
- [Directory Reference](#directory-reference)
- [File Reference](#file-reference)
  - [Configuration](#configuration)
  - [Integrations — Stage3 Builder](#integrations--stage3-builder)
  - [Integrations — penguins-incus-platform](#integrations--penguins-incus-platform)
  - [Packaging — Portage Overlay](#packaging--portage-overlay)
  - [Source — Package Manager](#source--package-manager)
  - [Source — Image Production](#source--image-production)
  - [Source — Installer (krill)](#source--installer-krill)
  - [Source — Supporting Classes](#source--supporting-classes)
  - [Documentation](#documentation)
- [Architecture Support](#architecture-support)
- [Quick Start](#quick-start)

---

## Overview

ChromiumOS is Gentoo-derived. Key differences from Debian/Arch-based distros:

| Aspect | ChromiumOS |
|---|---|
| Package manager | Portage (`emerge`) + Chromebrew (`crew`) |
| Library path | `/usr/lib64/` |
| initramfs builder | `dracut` |
| Installer | `krill` (TUI) — Calamares not available on standard builds |
| Partition layout | 12-partition CrOS GPT via `cgpt` |
| Boot | depthcharge (hardware) or GRUB EFI (generic/container) |
| Kernel cmdline extra | `cros_debug` |

---

## Supported Derivatives

Defined in [`conf/derivatives_chromiumos.yaml`]({{REPO_BLOB}}/conf/derivatives_chromiumos.yaml):

| Derivative | Notes |
|---|---|
| ChromiumOS / ChromeOS | Upstream reference builds |
| FydeOS / openFyde | Hardware boards: rpi4, rpi5, rk3588, rk3399, orangepi5 |
| ThoriumOS | Compiler-optimized Chromium fork |
| WayneOS | Community build |
| Brunch | ChromiumOS on generic x86 hardware |
| ChromiumOS-stage3 | Build-environment containers (this project) |

---

## Directory Reference

18 directories across configuration, integrations, packaging, and source.

### Configuration

| Directory | Purpose |
|---|---|
| [`conf/distros/chromiumos`]({{REPO_TREE}}/conf/distros/chromiumos) | Distro-level config: Calamares templates, krill settings |
| [`conf/distros/chromiumos/calamares`]({{REPO_TREE}}/conf/distros/chromiumos/calamares) | Calamares installer config for ChromiumOS |
| [`conf/distros/chromiumos/calamares/calamares-modules`]({{REPO_TREE}}/conf/distros/chromiumos/calamares/calamares-modules) | Custom Calamares modules (e.g. bootloaderspecification) |
| [`conf/distros/chromiumos/calamares/calamares-modules/bootloaderspecification`]({{REPO_TREE}}/conf/distros/chromiumos/calamares/calamares-modules/bootloaderspecification) | EFI bootloader specification module |
| [`conf/distros/chromiumos/calamares/modules`]({{REPO_TREE}}/conf/distros/chromiumos/calamares/modules) | Per-module YAML overrides for ChromiumOS Calamares |
| [`conf/wardrobes/chromiumos`]({{REPO_TREE}}/conf/wardrobes/chromiumos) | Wardrobe root — browser flavour definitions |
| [`conf/wardrobes/chromiumos/costumes`]({{REPO_TREE}}/conf/wardrobes/chromiumos/costumes) | All available browser costumes |
| [`conf/wardrobes/chromiumos/costumes/brave`]({{REPO_TREE}}/conf/wardrobes/chromiumos/costumes/brave) | Brave browser costume |
| [`conf/wardrobes/chromiumos/costumes/chromium`]({{REPO_TREE}}/conf/wardrobes/chromiumos/costumes/chromium) | Stock Chromium costume (default) |
| [`conf/wardrobes/chromiumos/costumes/custom`]({{REPO_TREE}}/conf/wardrobes/chromiumos/costumes/custom) | Custom browser via `--cros-browser-repo` |
| [`conf/wardrobes/chromiumos/costumes/thorium`]({{REPO_TREE}}/conf/wardrobes/chromiumos/costumes/thorium) | Thorium (Alex313031) costume |

### Integrations

| Directory | Purpose |
|---|---|
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3`]({{REPO_TREE}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3) | Stage3 tarball builder root |
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/.github/workflows`]({{REPO_TREE}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/.github/workflows) | CI workflows — builds stage3 tarballs on schedule |
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/boards`]({{REPO_TREE}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/boards) | Per-board build configs (reven, arm64-generic, rpi4, rpi5, rk3588…) |

### Packaging

| Directory | Purpose |
|---|---|
| [`packaging/chromiumos`]({{REPO_TREE}}/packaging/chromiumos) | Portage overlay + Chromebrew recipe root |
| [`packaging/chromiumos/chromebrew`]({{REPO_TREE}}/packaging/chromiumos/chromebrew) | Chromebrew (`crew`) package recipe |
| [`packaging/chromiumos/overlay`]({{REPO_TREE}}/packaging/chromiumos/overlay) | Gentoo/Portage overlay tree |
| [`packaging/chromiumos/overlay/app-misc/penguins-eggs`]({{REPO_TREE}}/packaging/chromiumos/overlay/app-misc/penguins-eggs) | `app-misc/penguins-eggs` ebuild and metadata |

---

## File Reference

49 files across configuration, integrations, packaging, source, and docs.

---

### Configuration

| File | Category | Purpose |
|---|---|---|
| [`conf/derivatives_chromiumos.yaml`]({{REPO_BLOB}}/conf/derivatives_chromiumos.yaml) | Config — Derivatives | Maps all ChromiumOS derivative IDs to `familyId: chromiumos`. Add new forks here without code changes. |
| [`conf/distros/chromiumos/README.md`]({{REPO_BLOB}}/conf/distros/chromiumos/README.md) | Config — Docs | Describes the ChromiumOS distro family, Gentoo heritage, and installer choice. |
| [`conf/distros/chromiumos/calamares/settings.yaml`]({{REPO_BLOB}}/conf/distros/chromiumos/calamares/settings.yaml) | Config — Installer | Calamares sequence for ChromiumOS: partition → unpackfs → dracut → bootloaderspecification. |
| [`conf/flavours/chromiumos.yaml`]({{REPO_BLOB}}/conf/flavours/chromiumos.yaml) | Config — Flavours | Registry of all browser flavours (chromium, thorium, brave, vanadium, cromite, openfyde). Defines install method, tarball filters, and ISO branding per flavour. |
| [`conf/wardrobes/chromiumos/index.yaml`]({{REPO_BLOB}}/conf/wardrobes/chromiumos/index.yaml) | Config — Wardrobe | Lists available costumes. Entry point for `eggs wardrobe wear --wardrobe chromiumos`. |
| [`conf/wardrobes/chromiumos/costumes/brave/chromiumos.yaml`]({{REPO_BLOB}}/conf/wardrobes/chromiumos/costumes/brave/chromiumos.yaml) | Config — Costume | Brave browser costume: tarball install from GitHub releases, arch-specific filters. |
| [`conf/wardrobes/chromiumos/costumes/chromium/chromiumos.yaml`]({{REPO_BLOB}}/conf/wardrobes/chromiumos/costumes/chromium/chromiumos.yaml) | Config — Costume | Stock Chromium costume: installed via `emerge www-client/chromium` or `crew install chromium`. |
| [`conf/wardrobes/chromiumos/costumes/custom/chromiumos.yaml`]({{REPO_BLOB}}/conf/wardrobes/chromiumos/costumes/custom/chromiumos.yaml) | Config — Costume | Custom browser costume: clones any git repo and runs its build script. |
| [`conf/wardrobes/chromiumos/costumes/thorium/chromiumos.yaml`]({{REPO_BLOB}}/conf/wardrobes/chromiumos/costumes/thorium/chromiumos.yaml) | Config — Costume | Thorium browser costume: tarball install from Alex313031/thorium releases. |

---

### Integrations — Stage3 Builder

The `chromiumos-stage3` component builds arch-agnostic ChromiumOS build-environment
containers. These containers include a full Portage tree and toolchain, enabling
`emerge`-based package management inside Incus containers.

| File | Category | Purpose |
|---|---|---|
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/README.md`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/README.md) | Integration — Docs | Board table, usage examples, requirements, overlay sources per board. |
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/build.sh`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/build.sh) | Integration — Builder | Main stage3 build script. Accepts `--board`, `--jobs`, `--output`. Produces `chromiumos-stage3-<board>-<release>.tar.xz`. |
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/.github/workflows/build.yml`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/.github/workflows/build.yml) | Integration — CI | GitHub Actions workflow. Builds `reven` (amd64) and `arm64-generic` tarballs on a weekly schedule and on push. |
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/boards/arm64-generic.conf`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/boards/arm64-generic.conf) | Integration — Board Config | Board config for generic arm64 (VMware/QEMU). Sets `ARCH`, `CHOST`, `BOOTSTRAP_URL`, openFyde overlay source. |
| [`integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/boards/reven.conf`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/boards/reven.conf) | Integration — Board Config | Board config for generic amd64 (`reven`). Uses upstream ChromiumOS overlay from `chromium.googlesource.com`. |

---

### Integrations — penguins-incus-platform

| File | Category | Purpose |
|---|---|---|
| [`integrations/penguins-incus-platform/unified-image-server/README.md`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/README.md) | Integration — Docs | Overview of the unified image server, including ChromiumOS image types. |
| [`integrations/penguins-incus-platform/unified-image-server/manifests/README.md`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/manifests/README.md) | Integration — Docs | Manifest format docs, including ChromiumOS image manifest structure. |
| [`integrations/penguins-incus-platform/unified-image-server/manifests/bin/build-chromiumos-image.sh`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/manifests/bin/build-chromiumos-image.sh) | Integration — Builder | Downloads a pre-built stage3 tarball and repackages it as an Incus-compatible image (`rootfs.tar.xz` + `metadata.tar.xz`). |
| [`integrations/penguins-incus-platform/unified-image-server/penguins-eggs/README.md`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/penguins-eggs/README.md) | Integration — Docs | How penguins-eggs runs inside the unified image server, including ChromiumOS specifics. |
| [`integrations/penguins-incus-platform/unified-image-server/penguins-eggs/conf/derivatives_chromiumos.yaml`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/penguins-eggs/conf/derivatives_chromiumos.yaml) | Integration — Config | Mirror of `conf/derivatives_chromiumos.yaml` for use inside the image server container. |
| [`integrations/penguins-incus-platform/unified-image-server/penguins-eggs/conf/flavours/chromiumos.yaml`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/penguins-eggs/conf/flavours/chromiumos.yaml) | Integration — Config | Mirror of `conf/flavours/chromiumos.yaml` for use inside the image server container. |
| [`integrations/penguins-incus-platform/unified-image-server/penguins-eggs/src/classes/pacman.d/chromiumos.ts`]({{REPO_BLOB}}/integrations/penguins-incus-platform/unified-image-server/penguins-eggs/src/classes/pacman.d/chromiumos.ts) | Integration — Source | Mirror of `src/classes/pacman.d/chromiumos.ts` bundled inside the image server container. |

---

### Packaging — Portage Overlay

| File | Category | Purpose |
|---|---|---|
| [`packaging/chromiumos/overlay/app-misc/penguins-eggs/metadata.xml`]({{REPO_BLOB}}/packaging/chromiumos/overlay/app-misc/penguins-eggs/metadata.xml) | Packaging — Portage | Gentoo package metadata for `app-misc/penguins-eggs`. Declares maintainer and describes ChromiumOS family support. |

---

### Source — Package Manager

`src/classes/pacman.d/chromiumos.ts` is the ChromiumOS package manager backend.
It abstracts over Portage (`emerge`), Chromebrew (`crew`), and `/var/db/pkg`.

| File | Category | Purpose |
|---|---|---|
| [`src/classes/pacman.d/chromiumos.ts`]({{REPO_BLOB}}/src/classes/pacman.d/chromiumos.ts) | Source — Pacman | ChromiumOS package manager class. Handles install/remove/check via emerge (stage3/cros_sdk) and crew (runtime). Includes variant detection, board detection, Portage capability checks, and EFI boot binary selection per arch. |
| [`src/classes/pacman.ts`]({{REPO_BLOB}}/src/classes/pacman.ts) | Source — Pacman | Main pacman dispatcher. Routes to `ChromiumOS` class when `familyId === 'chromiumos'`. |

---

### Source — Image Production

These files handle ChromiumOS-specific ISO/image creation.

| File | Category | Purpose |
|---|---|---|
| [`src/classes/ovary.d/cros_flavour.ts`]({{REPO_BLOB}}/src/classes/ovary.d/cros_flavour.ts) | Source — Ovary | Flavour builder. Applies a browser costume before ISO creation. Handles tarball download (thorium/brave), emerge install (chromium), and custom git-repo builds. |
| [`src/classes/ovary.d/cros_verity.ts`]({{REPO_BLOB}}/src/classes/ovary.d/cros_verity.ts) | Source — Ovary | dm-verity setup for ChromiumOS rootfs partitions (ROOT-A/ROOT-B). Generates verity hash tree and kernel cmdline parameters. |
| [`src/classes/ovary.d/produce.ts`]({{REPO_BLOB}}/src/classes/ovary.d/produce.ts) | Source — Ovary | Main produce pipeline. Contains ChromiumOS-specific branch: uses dracut initrd builder, sets `cros_debug` kernel param, applies flavour. |
| [`src/classes/distro.ts`]({{REPO_BLOB}}/src/classes/distro.ts) | Source — Distro | Distro detection. Maps ChromiumOS derivative IDs to `familyId: 'chromiumos'` via `derivatives_chromiumos.yaml`. |
| [`src/classes/diversions.ts`]({{REPO_BLOB}}/src/classes/diversions.ts) | Source — Diversions | Kernel parameter builder. Adds `cros_debug` to the standard dracut live boot params for the ChromiumOS family. |
| [`src/classes/tailor.ts`]({{REPO_BLOB}}/src/classes/tailor.ts) | Source — Tailor | Wardrobe/costume applicator. Handles ChromiumOS costume YAML structure and browser install steps. |
| [`src/commands/produce.ts`]({{REPO_BLOB}}/src/commands/produce.ts) | Source — Commands | `eggs produce` CLI entry point. Exposes `--cros-flavour` and `--cros-browser-repo` flags for ChromiumOS. |
| [`src/commands/export/pkg.ts`]({{REPO_BLOB}}/src/commands/export/pkg.ts) | Source — Commands | `eggs export pkg` command. Includes ChromiumOS Portage overlay export path. |
| [`src/commands/tools/repo.ts`]({{REPO_BLOB}}/src/commands/tools/repo.ts) | Source — Commands | `eggs tools repo` command. Configures the Portage overlay or Chromebrew tap for ChromiumOS. |
| [`src/commands/update.ts`]({{REPO_BLOB}}/src/commands/update.ts) | Source — Commands | `eggs update` command. Handles self-update via emerge or crew on ChromiumOS. |
| [`src/appimage/dependency-manager.ts`]({{REPO_BLOB}}/src/appimage/dependency-manager.ts) | Source — AppImage | AppImage dependency manager. Skips bundling Portage-managed libs on ChromiumOS. |

---

### Source — Installer (krill)

`krill` is the TUI installer used on ChromiumOS (Calamares is not available on
standard ChromiumOS builds). These files contain ChromiumOS-specific installer logic.

| File | Category | Purpose |
|---|---|---|
| [`src/krill/classes/sequence.d/cros_partition.ts`]({{REPO_BLOB}}/src/krill/classes/sequence.d/cros_partition.ts) | Source — krill | CrOS GPT partition layout via `cgpt`. Implements the 12-partition CrOS layout (STATE, KERN-A/B, ROOT-A/B, KERN-C/ROOT-C, OEM, RWFW, EFI-SYSTEM). Handles partition creation, kernel signing, and formatting. |
| [`src/krill/classes/sequence.d/bootloader_config.ts`]({{REPO_BLOB}}/src/krill/classes/sequence.d/bootloader_config.ts) | Source — krill | Bootloader configuration step. ChromiumOS branch installs GRUB EFI (generic boards) or configures depthcharge (hardware boards). |
| [`src/krill/classes/sequence.d/initramfs.ts`]({{REPO_BLOB}}/src/krill/classes/sequence.d/initramfs.ts) | Source — krill | initramfs rebuild step. ChromiumOS branch calls `dracut` inside the install target. |
| [`src/krill/classes/sequence.d/locale.ts`]({{REPO_BLOB}}/src/krill/classes/sequence.d/locale.ts) | Source — krill | Locale configuration step. ChromiumOS branch uses `/etc/locale.gen` under Portage conventions. |
| [`src/krill/classes/sequence.d/locale_cfg.ts`]({{REPO_BLOB}}/src/krill/classes/sequence.d/locale_cfg.ts) | Source — krill | Locale config file writer. Handles ChromiumOS-specific locale paths. |

---

### Source — Supporting Classes

| File | Category | Purpose |
|---|---|---|
| [`src/classes/bleach.ts`]({{REPO_BLOB}}/src/classes/bleach.ts) | Source — Bleach | System cleanup before ISO snapshot. ChromiumOS branch removes Portage build artifacts and stage3 temp files. |
| [`src/classes/incubation/fisherman.ts`]({{REPO_BLOB}}/src/classes/incubation/fisherman.ts) | Source — Incubation | Live system setup. ChromiumOS branch configures the live user environment under ChromiumOS conventions. |
| [`src/classes/incubation/fisherman-helper/packages.ts`]({{REPO_BLOB}}/src/classes/incubation/fisherman-helper/packages.ts) | Source — Incubation | Package list helper. Returns ChromiumOS-specific packages to install/exclude during live system preparation. |
| [`src/classes/incubation/incubator.ts`]({{REPO_BLOB}}/src/classes/incubation/incubator.ts) | Source — Incubation | Incubator orchestrator. Dispatches to ChromiumOS-specific incubation steps. |
| [`src/classes/locales.ts`]({{REPO_BLOB}}/src/classes/locales.ts) | Source — Locales | Locale detection and generation. ChromiumOS branch reads from `/etc/locale.gen` under Portage. |
| [`src/classes/pxe.ts`]({{REPO_BLOB}}/src/classes/pxe.ts) | Source — PXE | PXE boot configuration. ChromiumOS branch sets the correct kernel cmdline including `cros_debug`. |

---

### Documentation

| File | Category | Purpose |
|---|---|---|
| [`README.md`]({{REPO_BLOB}}/README.md) | Docs — Root | Project README. References ChromiumOS as a supported family. |
| [`GEMINI.md`]({{REPO_BLOB}}/GEMINI.md) | Docs — AI | AI assistant context file. Includes ChromiumOS family notes for Gemini. |
| [`integrations/ARCHITECTURE.md`]({{REPO_BLOB}}/integrations/ARCHITECTURE.md) | Docs — Architecture | Integration architecture overview. Describes how ChromiumOS stage3 containers fit into the broader system. |
| [`integrations/INTEGRATIONS.md`]({{REPO_BLOB}}/integrations/INTEGRATIONS.md) | Docs — Integrations | Integration catalogue. Lists ChromiumOS image server integration. |
| [`integrations/PROJECT-CATALOG.md`]({{REPO_BLOB}}/integrations/PROJECT-CATALOG.md) | Docs — Catalog | Project catalog. Includes ChromiumOS-stage3 and related components. |
| [`integrations/README.md`]({{REPO_BLOB}}/integrations/README.md) | Docs — Integrations | Integrations root README. Covers ChromiumOS image server setup. |
| [`package.json`]({{REPO_BLOB}}/package.json) | Config — Project | Project manifest. References ChromiumOS in supported distro metadata. |

---

## Architecture Support

ChromiumOS support in penguins-eggs is architecture-agnostic. The same codebase
handles all four architectures:

| Arch | EFI binary | Shim | Notes |
|---|---|---|---|
| amd64 (x64) | `bootx64.efi` | `shimx64.efi` | Primary target, `reven` board |
| arm64 | `bootaa64.efi` | `shimaa64.efi` | openFyde boards, `arm64-generic` |
| i386 (ia32) | `bootia32.efi` | `shimia32.efi` | Legacy 32-bit EFI |
| riscv64 | `bootriscv64.efi` | none | GRUB used directly — no shim available |

Architecture selection is handled generically in `make-efi.ts`. No
ChromiumOS-specific code is needed per arch beyond the EFI binary name.

---

## Quick Start

### Produce a ChromiumOS ISO (default Chromium browser)

```bash
eggs produce --cros-flavour chromium
```

### Produce with Thorium browser

```bash
eggs produce --cros-flavour thorium
```

### Produce with a custom browser

```bash
eggs produce --cros-flavour custom --cros-browser-repo https://github.com/your/browser
```

### Build a stage3 container (amd64)

```bash
cd integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3
sudo ./build.sh --board reven
```

### Build a stage3 container (arm64)

```bash
sudo apt-get install qemu-user-static
sudo ./build.sh --board arm64-generic
```

### Build an Incus image from a pre-built stage3

```bash
cd integrations/penguins-incus-platform/unified-image-server/manifests/bin
./build-chromiumos-image.sh --board reven --release R146
```

### Apply a wardrobe costume manually

```bash
eggs wardrobe wear --costume thorium --wardrobe chromiumos
```

---

## Summary

| Category | Directories | Files |
|---|---|---|
| Configuration | 11 | 9 |
| Integrations | 4 | 12 |
| Packaging | 3 | 1 |
| Source — Package Manager | — | 2 |
| Source — Image Production | — | 11 |
| Source — Installer (krill) | — | 5 |
| Source — Supporting Classes | — | 6 |
| Documentation | — | 7 |
| **Total** | **18** | **49** |
