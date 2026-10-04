Unofficial recovery for **SM-M115F** with FBE decryption for tested custom-ROM configurations and Advanced > Essentials.

## Downloads

- **TWRP-3.7.1_12-quokka-m11q-20261004-r2.tar** — Odin AP package with one root-level recovery.img.
- **TWRP-3.7.1_12-quokka-m11q-20261004-r2.img** — the identical 64 MiB recovery image; select Recovery when installing an image.

## Changes

- **r2: optional backups for Prepare AVB, Keep TWRP and Magisk.** Each operation offers **Back up first** or **Skip backup**, followed by swipe confirmation. Skip backup works without SD/OTG and does not disable partition checks or write verification.
- In Skip backup mode, this tool saves no persistent backup. Boot/vbmeta originals held in memory can only support immediate rollback during the operation; no saved copy remains after reboot. Keep TWRP retains restore files by renaming them in place.

- Expanded Essentials: Keep TWRP, official stable Magisk 30.7, Android SELinux controls, root-module controls, boot and Sec EFS backups, RW mounts and diagnostics.
- Separate AVB Status / Prepare AVB with optional current boot and vbmeta backups before flag changes. This action preserves Data encryption.
- Fix Samsung bootloader-state detection when optional lock properties are omitted.
- Improve Essentials spacing and Advanced navigation, restore F2FS tools and order Advanced Wipe entries.
- Add logical and physical Install Image targets with live capacity, mount/read-only and raw/sparse write checks.
- Use generic public interface and installer text; preserve the working decryption stack and fix duplicate recovery-library installers.

## Installation

Requires an unlocked bootloader. In Odin, select the TAR in **AP**, disable Auto Reboot, flash, then boot directly into recovery after PASS. The TAR contains only recovery.img; it does not flash boot or vbmeta.

## Stock ROM: keep TWRP after reboot

On stock ROM with AVB verification enabled, **Prepare AVB must come before Keep TWRP**. The following sequence was confirmed on the tested device:

1. Flash the Odin TAR with **Auto Reboot disabled**, then boot directly into TWRP. Complete the steps below before booting Android.
2. If you want backups, open **Advanced > Essentials > Select Storage** and select writable storage, such as **Micro SD** or **USB OTG**. If you have neither and stock internal storage is unavailable, select **Skip backup** when each operation asks.
3. Open **Boot / AVB > AVB / DM-Verity > Status**. If verification is enabled, select **Prepare AVB**, choose **Back up first** or **Skip backup**, then swipe to confirm. Back up first saves the current **vbmeta and boot backups** to the selected storage. Both choices disable the current vbmeta verification/hashtree flags and verify the write. If the flags are already disabled, no additional preparation is needed.
4. Return to **Essentials > Keep TWRP**, choose your backup option and swipe to confirm. Check the log for successful disabling of the stock recovery restore files; Back up first also saves copies to the selected storage.
5. Reboot to **System**, then return to **Recovery** and confirm quokka TWRP is still present.

The bootloader must already be unlocked. Prepare AVB is a separate confirmed action, not an automatic part of Keep TWRP. Restore matching complete firmware when returning to verified stock after boot/ROM changes.

Magisk is optional and separate from recovery persistence. Choose Back up first or Skip backup before installation; reboot manually and confirm root in Android afterward.

## FBE decryption and encryption-mode switches

**Stock ROM decryption is not supported by this recovery.** The successful FBE decryption tests apply to the tested custom-ROM configurations, not stock ROM.

**Enable FBE / Disable FBE are not supported on stock ROM.**

These switches change the ROM's encryption configuration; they do not decrypt existing encrypted files. On a compatible ROM, switching encryption mode requires a separate manual **Format Data**, which erases internal storage.

**Prepare AVB preserves Data encryption and does not format Data.** It does not enable stock ROM decryption. The stock Keep TWRP workflow does not require Disable FBE or Format Data.

## Validation

- User confirmed the r2 optional-backup build works in device testing.

- User confirmed recovery reinstall of Magisk 30.7 after removing the previous installation.
- User confirmed Keep TWRP persists after reboot and Android SELinux Enforcing/Permissive controls work.
- Pattern/PIN/password FBE decryption on tested custom-ROM configurations and Odin installation were previously confirmed; the decryption input bytes remain unchanged in this release. Stock ROM decryption is not supported.
- Packed files, ELF integrity, source/payload hashes, interface geometry, recovery AVB signature/hash and dynamic symbols pass. TAR payload matches the tested IMG exactly.
- Other backup/restore tools and the additional image-writing targets have not all been tested on the phone. Image flashing does not automatically resize partitions.

## Sources and guide

See [the device guide](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/blob/fbe-reconstruction/ESSENTIALS.md) and [build inputs](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/blob/fbe-reconstruction/BUILD_INFO-20261004-r2.json) for exact source revisions. Source changes are split into logical commits for cherry-picking.

## Credits

- Thanks to @goldfish07 for the TWRP device source.