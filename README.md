# Surface Duo 2 QGKI Kernel

Unofficial Linux 5.4 QGKI kernel work for the Microsoft Surface Duo 2, based on Microsoft's `surfaceduo2/11/2023.501.541` source release.

This repository keeps three tested development points:

| Branch | Purpose |
| --- | --- |
| `duo2-bootable-stock` | Stock-derived QGKI baseline with the changes required to boot the locally built kernel. |
| `duo2-rksu-only` | Bootable baseline plus current rsuntk KernelSU manual hooks and the reboot driver-FD hook. |
| `duo2-rksu-susfs` | Complete rksu build plus the Surface Duo 2 SUSFS v1.5.5 integration. |
| `main` | Currently follows `duo2-rksu-susfs`. |

## Tested baseline

- Device: Microsoft Surface Duo 2
- Kernel base: Linux 5.4 QGKI
- Microsoft source branch: `surfaceduo2/11/2023.501.541`
- Build target: `lahaina-qgki_defconfig`
- Build type: `user`
- Expected kernel release: `5.4.233-qgki-c1-p-0_0_0`

The stock-derived bootable baseline disables forced kernel module signing and removes the dirty-tree `+` suffix from the release string. These are the settings used by the tested build.

## Clone

Clone with submodules:

```bash
git clone --recurse-submodules \
  https://github.com/dtingley11/SurfaceDuo2-QGKI-Kernel.git
```

For an existing clone:

```bash
git submodule update --init --recursive
```

The `main` and `duo2-rksu-susfs` branches use this customized rksu submodule:

```text
https://github.com/dtingley11/rksu-duo2-susfs.git
```

## Branch selection

Complete rksu + SUSFS build:

```bash
git switch duo2-rksu-susfs
git submodule update --init --recursive
```

rksu without SUSFS:

```bash
git switch duo2-rksu-only
git submodule update --init --recursive
```

Stock-derived bootable baseline:

```bash
git switch duo2-bootable-stock
```

## Build environment

This repository is the kernel tree used inside Microsoft's complete Surface Duo 2 Android build environment. The tested build script is located outside this repository at:

```text
device_build/kernel/kernel-build.sh
```

Example parent workspace layout:

```text
Surface_Duo2/
├── device_build/
│   └── kernel/
│       └── kernel-build.sh
└── kernel/
    └── msm-5.4/        # this repository
```

From the parent `Surface_Duo2` workspace, run:

```bash
cd /path/to/Surface_Duo2

./device_build/kernel/kernel-build.sh \
  -c lahaina-qgki_defconfig \
  -tb user \
  -p "$PWD"
```

The exact output location depends on the surrounding Microsoft build tree and script revision. Confirm that the resulting kernel reports:

```text
5.4.233-qgki-c1-p-0_0_0
```

Do not treat this kernel repository as a standalone Android build environment.

## Boot testing

Test a newly packed boot image without flashing it first:

```bash
adb reboot bootloader
fastboot boot boot.img
```

Only flash after confirming that Android boots, root works, and required vendor modules load correctly.

A failed or incompatible kernel can prevent the device from booting. Keep a known-good boot image and working fastboot access available.

## rksu integration

The rksu branches are based on rsuntk KernelSU commit:

```text
648e5988cf421172769f80ce07f86331b548c053
```

The integration uses manual hooks for:

- `execveat`
- `faccessat`
- VFS reads
- stat and fstat operations

It also adds the reboot syscall driver-FD hook required by current `ksud` feature detection. On the tested build:

```text
ksud debug version                 -> Kernel Version: 32473
ksud feature check kernel_umount   -> supported
```

## SUSFS v1.5.5

The complete branch contains a Surface Duo 2 Linux 5.4 port of SUSFS v1.5.5.

Enabled features:

```text
CONFIG_KSU_SUSFS_SUS_MOUNT
CONFIG_KSU_SUSFS_SUS_KSTAT
CONFIG_KSU_SUSFS_TRY_UMOUNT
CONFIG_KSU_SUSFS_SPOOF_UNAME
CONFIG_KSU_SUSFS_ENABLE_LOG
CONFIG_KSU_SUSFS_HIDE_KSU_SUSFS_SYMBOLS
CONFIG_KSU_SUSFS_OPEN_REDIRECT
```

Intentionally disabled or unsupported in this port:

```text
CONFIG_KSU_SUSFS_HAS_MAGIC_MOUNT
CONFIG_KSU_SUSFS_SUS_PATH
CONFIG_KSU_SUSFS_AUTO_ADD_SUS_KSU_DEFAULT_MOUNT
CONFIG_KSU_SUSFS_AUTO_ADD_SUS_BIND_MOUNT
CONFIG_KSU_SUSFS_SUS_OVERLAYFS
CONFIG_KSU_SUSFS_AUTO_ADD_TRY_UMOUNT_FOR_BIND_MOUNT
CONFIG_KSU_SUSFS_SPOOF_CMDLINE_OR_BOOTCONFIG
CONFIG_KSU_SUSFS_SUS_SU
```

The disabled features are not merely hidden defaults. Some depend on kernel paths or rksu implementations that were not present or not validated for this Surface Duo 2 tree.

Runtime validation on the tested build returned:

```text
SUSFS version: v1.5.5
SUSFS variant: GKI
```

## Tags

Known-good points are marked with annotated tags:

```text
duo2-qgki-stock-bootable
duo2-qgki-rksu-only-working
duo2-qgki-rksu-susfs-v1.5.5-working
```

## Important notes

- This project is unofficial and is not affiliated with or endorsed by Microsoft.
- It is specifically tested against the Surface Duo 2 source and firmware combination described above.
- Do not assume compatibility with the original Surface Duo, unrelated Qualcomm devices, or a different Surface Duo 2 firmware release.
- Do not restore old KernelSU, SUSFS, or module data blindly across kernel versions.
- Initial testing should always use `fastboot boot` rather than immediately flashing.

## Credits

- Microsoft for publishing the Surface Duo kernel source.
- The Android and Linux kernel projects.
- rsuntk for KernelSU/rksu.
- simonpunk for SUSFS.

## License

The Linux kernel source in this repository retains its existing licensing. See `COPYING` and the SPDX identifiers in individual source files. The kernel tree is primarily provided under GPL-2.0 with the Linux syscall note, while some files may use other compatible licenses as documented in the source tree.
