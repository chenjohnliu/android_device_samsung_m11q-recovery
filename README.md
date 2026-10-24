# TWRP 3.7.1_12-quokka for Samsung Galaxy M11

English documentation | [Traditional Chinese guide](ESSENTIALS.zh-TW.md)

Device: m11q / SM-M115F. Includes FBE decryption for fileencryption=ice on tested custom-ROM configurations,
Advanced > Essentials and additional Install Image targets.

The 2026-10-04 r2 release preserves the exact user-tested recovery bytes. Download
IMG or Odin AP TAR from [the r2 release](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/releases/tag/3.7.1_12-quokka-20261004-r2). GitHub provides asset digests. The TAR
contains only recovery.img and is byte-identical to the standalone IMG.

## Current source update: fastboot1 (2026-10-24)

The current source starts the bundled Health HAL and uses Samsung fastboot USB
identification. Recovery boot, fastbootd connection with the existing Samsung
driver, logical partition size queries and return to Recovery passed device
testing. Image flashing retains the existing capacity checks; automatic resizing
and GSI Tools are not included. See [build notes](RELEASE_NOTES-20261024-fastboot1.md)
and [fastboot instructions](ESSENTIALS.md#fastbootd). The public r2 release assets
above are unchanged; this commit does not publish new image downloads.

## Build sources

Use the device branch fbe-reconstruction and the exact companion revisions below:

- `bootable/recovery`: [c31a88d333ab](https://github.com/chenjohnliu/android_bootable_recovery/commit/c31a88d333ab80aa25775c165f0e604d0b00aab5) (`m11q-fbe-essentials`).
- `device/samsung/m11q`: [d9b5f49dfdc4](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/commit/d9b5f49dfdc4f37d9e48e95abe6effe01895cecc) (`fbe-reconstruction`).
- `device/qcom/twrp-common`: [70ecbe3018bf](https://github.com/chenjohnliu/android_device_qcom_twrp-common/commit/70ecbe3018bf600de7d4cc452b66a543909d0673) (`m11q-32bit-services`).
- `system/core`: [9919f11d0c00](https://github.com/chenjohnliu/android_system_core/commit/9919f11d0c00885006d2ca22ef034ae8eaf5bff0) (`m11q-fbe-diagnostics`).

The checked-in manifests/twrp12.1-base.xml pins the other base checkouts.
Override the four checkouts above before building. Check FBE_INPUTS.sha256
from this device directory; it records the kernel and every staged root input.
Do not mix vendor runtime versions or change crypto inputs without new evidence.

```sh
export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh
lunch twrp_m11q-eng
mka -j4 recoveryimage
```

Verify the packed image, including full ELF files and native library ownership,
before flashing. The separate recovery-library fix prevents duplicate relink
writers from truncating native libraries.

## Essentials

See [ESSENTIALS.md](ESSENTIALS.md). Magisk 30.7 is the unchanged official signed
APK with a separate wrapper that forces boot-only installation and retains
encryption/verity. AVB preparation is a distinct, explicitly confirmed action
with optional boot/vbmeta backups that changes only current verification flags.
Prepare AVB, Keep TWRP and Magisk offer Back up first or explicit Skip backup,
then swipe confirmation. Skip mode needs no SD/OTG and saves no persistent backup.
Enable/Disable FBE switches require a manual Format Data and are independent
of recovery's normal FBE decryption.

## Stock ROM and FBE support

Stock ROM decryption is not supported. Enable FBE / Disable FBE are not supported
on stock ROM. Successful pattern/PIN/password decryption tests apply to the tested
custom-ROM configurations.

On stock, choose backup storage or explicit Skip backup, Prepare AVB when verification is enabled, then
Keep TWRP before booting Android. Check recovery persistence after reboot.
See [release notes](RELEASE_NOTES-20261004-r2.md) and [the guide](ESSENTIALS.md)
for the full sequence. This workflow does not require Disable FBE or Format Data.

## Validation

User confirmed the r2 optional-backup build works in device testing.

User confirmed pattern, PIN and password decryption on tested custom ROMs, Odin installation,
Android Enforcing/Permissive, stock recovery persistence and Magisk 30.7
reinstallation from recovery. The latest image's crypto bytes match the
previously validated inputs. Other tools and image-write targets still have
individual runtime checks pending; see the guide for exact behavior.
