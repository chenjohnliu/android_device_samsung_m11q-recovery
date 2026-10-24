# TWRP 3.7.1_12-quokka for Samsung Galaxy M11

English documentation | [Traditional Chinese guide](ESSENTIALS.zh-TW.md)

Device: m11q / SM-M115F. Includes FBE decryption for fileencryption=ice on tested custom-ROM configurations,
Advanced > Essentials and additional Install Image targets.

Download the **quokka r3 (2026-10-24)** IMG or Odin AP TAR from
[the r3 release](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/releases/tag/3.7.1_12-quokka-r3). The TAR contains only recovery.img and its payload
is identical to the standalone 64 MiB IMG. Runtime version: 3.7.1_12-quokka-r3.

## Changes in r3

- GSI Tools: separately inspect allocation, delete Product and grow System to
  a validated raw/sparse GSI size, with verified metadata backup and rollback.
- Native Backup > Sec EFS replaces the redundant Essentials raw-backup shortcut.
- Magisk > Root Modules groups the unchanged Magisk tools in one submenu.
- Working fastbootd Health startup/Samsung USB IDs, complete English resources,
  and a production-resolver style/font guard for the GSI picker.

See [r3 notes](RELEASE_NOTES-20261024-r3.md), the
[GSI guide](ESSENTIALS.md#gsi-tools), and
[build inputs](BUILD_INFO-20261024-r3.json). Historical r2/source-update notes
remain available separately. The user confirmed a successful GSI workflow using
the local predecessor with the same feature sources; the formal rebuild has not
been separately boot-tested. Native Sec EFS restore, module rescue and other
individual tools retain their documented validation limits.

## Build sources

Use the device branch fbe-reconstruction and the exact companion revisions below:

- `bootable/recovery`: [86d8677064ff](https://github.com/chenjohnliu/android_bootable_recovery/commit/86d8677064ff63fe329318d90e0e01ffcdc7d46c) (`m11q-fbe-essentials`).
- `device/samsung/m11q`: [ea3e99958f71](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/commit/ea3e99958f7105d415f0cddf112a178bdf7b5019) (`fbe-reconstruction`; exact r3 code/build revision, followed by documentation-only commits).
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
See [release notes](RELEASE_NOTES-20261024-r3.md) and [the guide](ESSENTIALS.md)
for the full sequence. This workflow does not require Disable FBE or Format Data.

## Validation

User confirmed the r2 optional-backup build works in device testing.

User confirmed pattern, PIN and password decryption on tested custom ROMs, Odin installation,
Android Enforcing/Permissive, stock recovery persistence and Magisk 30.7
reinstallation from recovery. The latest image's crypto bytes match the
previously validated inputs. Other tools and image-write targets still have
individual runtime checks pending; see the guide for exact behavior.
