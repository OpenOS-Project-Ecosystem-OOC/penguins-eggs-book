# Chapter 17

# ChromiumOS support

The `all-features` branch adds full ChromiumOS family support to penguins-eggs.
This chapter covers what ChromiumOS support means in practice, how to produce
ChromiumOS ISOs, and how the architecture-agnostic design works across amd64,
arm64, i386, and riscv64.

---

## What is the ChromiumOS family?

ChromiumOS is Gentoo-derived. It uses Portage (`emerge`) for package management,
dracut for initramfs, and a 12-partition CrOS GPT layout managed by `cgpt`.

penguins-eggs supports the full ChromiumOS derivative family:

| Derivative | Notes |
|---|---|
| ChromiumOS / ChromeOS | Upstream reference builds |
| FydeOS / openFyde | Hardware boards: rpi4, rpi5, rk3588, rk3399, orangepi5 |
| ThoriumOS | Compiler-optimized Chromium fork |
| WayneOS | Community build |
| Brunch | ChromiumOS on generic x86 hardware |
| ChromiumOS-stage3 | Build-environment containers (see below) |

Detection is automatic — eggs reads `/etc/lsb-release` and `/etc/os-release`
to identify the derivative and map it to `familyId: chromiumos`.

---

## Key differences from Debian/Arch-based distros

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

## Browser flavours

ChromiumOS images are built around a Chromium-based browser. eggs calls these
"flavours". Each flavour is a wardrobe costume in `conf/wardrobes/chromiumos/`.

| Flavour | Browser | Install method |
|---|---|---|
| `chromium` | Stock Chromium (default) | `emerge www-client/chromium` or `crew install chromium` |
| `thorium` | Thorium (Alex313031) — compiler-optimized | Pre-built tarball from GitHub releases |
| `brave` | Brave — privacy-focused | Pre-built tarball from GitHub releases |
| `custom` | Any Chromium fork | Clone git repo and run its build script |

---

## Producing a ChromiumOS ISO

### Default (Chromium browser)

```bash
sudo eggs produce --cros-flavour chromium
```

### With Thorium browser

```bash
sudo eggs produce --cros-flavour thorium
```

### With Brave browser

```bash
sudo eggs produce --cros-flavour brave
```

### With a custom browser

```bash
sudo eggs produce --cros-flavour custom --cros-browser-repo https://github.com/your/browser
```

The `--cros-flavour` flag applies the corresponding wardrobe costume before
ISO creation. The ISO is branded with the flavour name in the volume ID and
filename (e.g. `thoriumos-amd64-2026.04.iso`).

---

## Architecture support

ChromiumOS support in penguins-eggs is architecture-agnostic. The same
codebase handles all four architectures:

| Arch | EFI binary | Shim | Notes |
|---|---|---|---|
| amd64 (x64) | `bootx64.efi` | `shimx64.efi` | Primary target |
| arm64 | `bootaa64.efi` | `shimaa64.efi` | openFyde boards |
| i386 (ia32) | `bootia32.efi` | `shimia32.efi` | Legacy 32-bit EFI |
| riscv64 | `bootriscv64.efi` | none | GRUB used directly — no shim available |

Architecture selection is handled generically in `make-efi.ts`. No
ChromiumOS-specific code is needed per arch beyond the EFI binary name.

---

## The installer: krill

Calamares is not available on standard ChromiumOS builds. `krill` — the
penguins-eggs TUI installer — is the primary installer for this family.

krill includes ChromiumOS-specific steps:

- **CrOS partition layout** — creates the 12-partition GPT using `cgpt`:

  | Partition | Label | Type |
  |---|---|---|
  | 1 | STATE | ext4 — user data and `/usr/local` |
  | 2 | KERN-A | Signed kernel (depthcharge loads this) |
  | 3 | ROOT-A | rootfs A, dm-verity protected |
  | 4 | KERN-B | Fallback kernel |
  | 5 | ROOT-B | Fallback rootfs |
  | 6 | KERN-C | miniOS/recovery kernel |
  | 7 | ROOT-C | miniOS/recovery rootfs |
  | 8 | OEM | OEM customization |
  | 9–10 | reserved | Alignment |
  | 11 | RWFW | Read-write firmware |
  | 12 | EFI-SYSTEM | FAT32 EFI system partition |

- **dracut initramfs** — rebuilt inside the install target after rootfs copy
- **Bootloader** — GRUB EFI on generic boards; depthcharge on hardware boards
- **Locale** — uses `/etc/locale.gen` under Portage conventions

---

## Package management

ChromiumOS uses two package managers depending on the environment:

| Environment | Package manager | When used |
|---|---|---|
| stage3 container / cros_sdk chroot | Portage (`emerge`) | Full Portage tree available |
| Runtime ChromeOS/ChromiumOS | Chromebrew (`crew`) | No Portage tree |

eggs detects which is available via `hasPortage()` and `hasChromebrew()` and
prefers `emerge` when both are present.

Package installation example (eggs handles this automatically):

```bash
# In a stage3 container
emerge --ask=n www-client/chromium

# On a runtime ChromeOS system
crew install chromium
```

---

## ChromiumOS stage3 containers

The `integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3/`
directory contains an arch-agnostic ChromiumOS stage3 builder. Stage3 containers
include a full Portage tree and toolchain, enabling `emerge`-based package
management inside Incus containers.

### Supported boards

| Board | Arch | Notes |
|---|---|---|
| `reven` | amd64 | Generic x86_64 — suitable for containers |
| `arm64-generic` | arm64 | Generic arm64 (VMware/QEMU) — suitable for containers |
| `rpi4` | arm64 | Raspberry Pi 4B/400 |
| `rpi5` | arm64 | Raspberry Pi 5 |
| `rk3588` | arm64 | ROCK 5B, Orange Pi 5, Firefly RK3588 |
| `rk3399` | arm64 | Rock Pi 4B, ROCK 4C+ |
| `orangepi5` | arm64 | Orange Pi 5/5B/5 Plus |

### Building a stage3 container

```bash
cd integrations/penguins-incus-platform/unified-image-server/chromiumos-stage3

# Generic amd64 (default)
sudo ./build.sh

# Generic arm64 (requires qemu-user-static on x86_64 hosts)
sudo apt-get install qemu-user-static
sudo ./build.sh --board arm64-generic

# Raspberry Pi 4
sudo ./build.sh --board rpi4

# Rockchip RK3588 family
sudo ./build.sh --board rk3588
```

Output: `chromiumos-stage3-<board>-<release>.tar.xz`

Pre-built tarballs for `reven` and `arm64-generic` are published via GitHub
Actions on a weekly schedule.

### Building an Incus image from a pre-built stage3

```bash
cd integrations/penguins-incus-platform/unified-image-server/manifests/bin
./build-chromiumos-image.sh --board reven --release R146
```

This downloads the stage3 tarball and repackages it as an Incus-compatible
image (`rootfs.tar.xz` + `metadata.tar.xz`).

---

## Wardrobe costumes

Costumes are applied with `eggs wardrobe wear` before producing an ISO:

```bash
# List available ChromiumOS costumes
eggs wardrobe list --wardrobe chromiumos

# Apply the Thorium costume manually
eggs wardrobe wear --costume thorium --wardrobe chromiumos

# Then produce the ISO
sudo eggs produce
```

Or in one step with `--cros-flavour`:

```bash
sudo eggs produce --cros-flavour thorium
```

---

## Portage overlay

A Gentoo/Portage overlay for penguins-eggs is included at
`packaging/chromiumos/overlay/app-misc/penguins-eggs/`. This allows
installation via `emerge` in stage3 containers and cros_sdk chroots:

```bash
# Add the overlay (in a stage3 container or cros_sdk chroot)
eselect repository add penguins-eggs git https://github.com/Interested-Deving-1896/penguins-eggs.git
emaint sync -r penguins-eggs

# Install
emerge --ask app-misc/penguins-eggs
```

A Chromebrew recipe is also available at `packaging/chromiumos/chromebrew/`
for runtime ChromeOS/ChromiumOS environments:

```bash
crew install penguins-eggs
```

---

## Reference documentation

Full directory and file reference for all ChromiumOS-specific code.
The reference docs are automatically synced from `penguins-eggs/docs/chromiumos/`
into this book whenever they change:

- [chromiumos/README.md](chromiumos/README.md) — 18 directories and 49 files, grouped by category (synced copy, always current)
- [chromiumos/README.pdf](chromiumos/README.pdf) — PDF version (synced copy)

Source in penguins-eggs:

- [docs/chromiumos/README.md](https://github.com/Interested-Deving-1896/penguins-eggs/blob/all-features/docs/chromiumos/README.md)
- [docs/chromiumos/README.pdf](https://github.com/Interested-Deving-1896/penguins-eggs/blob/all-features/docs/chromiumos/README.pdf)
