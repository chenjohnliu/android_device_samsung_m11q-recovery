# quokka Essentials Guide

English | [Traditional Chinese](ESSENTIALS.zh-TW.md)

For the Samsung Galaxy M11 (`m11q`). Open **Advanced > Essentials**. Backups use the destination chosen in **Select Storage**, such as Micro SD or USB OTG. They are saved under `TWRP/Essentials/<device-serial>/<operation-and-time>/` with verification information. Write operations require swipe confirmation. These tools do not automatically reboot or format the device.

Release **20261004-r2** adds backup choices and has passed user device testing. **Prepare AVB**, **Keep TWRP**, and **Magisk** each offer **Back up first** or **Skip backup**. Skipping a backup still requires **Swipe without Backup**. If no writable storage is available, Skip backup works without SD or OTG.

Skip backup omits this tool's persistent backup. Device, bootloader, image, and write verification checks remain enabled. Original boot/vbmeta bytes held in memory can only support immediate rollback during the operation; there is no saved copy after reboot. Matching stock firmware may be needed for recovery. Keep TWRP retains the restore files by renaming them on their original partition rather than deleting them. Other backup, export, and restore tools still use the selected storage.

## Tools

| Menu | Function |
| --- | --- |
| Disable FBE / Enable FBE | **Not supported on stock ROM.** Changes the encryption configuration only on compatible custom ROMs. Back up your data and manually Format Data when switching modes. This does not decrypt existing data. |
| Keep TWRP | Finds stock recovery restore files, optionally backs them up, and disables them by renaming them. Hashtree verification must be disabled before ROM file changes. Reports when no supported files are found; it does not unlock KG. |
| Magisk 30.7 | Uses the bundled official stable version. Optionally backs up the current boot image, then installs into boot with a supported ramdisk. Preserves data encryption and verity settings. |
| Android SELinux | Changes Android boot parameters for Enforcing / Permissive. Restore Setting reverses the parameters changed by this tool. Each operation retains a boot backup. |
| Mount RW | Mounts system, vendor, product, and odm as writable for the current recovery session. Checks the actual result and reports failures. |
| Root Modules | Lists or disables one/all modules and saves their previous state. Choose Module selects `module.prop`; Restore States selects `modules.state`. Requires decrypted, writable Data. Unsupported root injection methods are rejected. |
| Boot / AVB | Backs up/restores boot and provides the AVB / DM-Verity submenu. Before restoring, validates the format, checksum, and ROM fingerprint, then backs up the current boot image. |
| Backup EFS | Backs up the raw `sec_efs` image. `efs` is a separate partition and is not included in this shortcut. |
| Diagnostics | Exports the recovery version, SELinux state, crypto services, boot parameters, mounts, and logs. |

## Stock ROM: keep TWRP after reboot

1. Flash the TAR in Odin **AP** with **Auto Reboot disabled**. Boot directly into TWRP afterward; do not start Android yet.
2. Open **Advanced > Essentials > Select Storage** and choose Micro SD or USB OTG for backups. If writable storage is unavailable, explicitly choose **Skip backup** for each supported operation.
3. Open **Boot / AVB > AVB / DM-Verity > Status**. If verification is enabled, run **Prepare AVB**, select **Back up first** or **Skip backup**, then swipe to confirm and wait for write verification. Back up first saves boot/vbmeta before changes.
4. Return to Essentials and run **Keep TWRP**. Choose the backup option, swipe to confirm, and check that the log reports disabling the stock recovery restore files.
5. Reboot to System, then return to Recovery and confirm quokka TWRP is still present.

The bootloader must already be unlocked. Prepare AVB is a separate, explicitly confirmed operation. Installing Magisk and keeping recovery are separate functions.

## FBE support

**This recovery does not support stock ROM decryption. Enable FBE / Disable FBE are also not supported on stock ROM.** Confirmed pattern/PIN/password decryption tests apply only to the tested custom-ROM configurations.

Switching the encryption configuration on a compatible ROM requires manual **Format Data**, which erases internal storage. Prepare AVB preserves Data encryption and does not format Data. It does not enable stock ROM decryption. The stock Keep TWRP workflow does not require Disable FBE or Format Data.

## AVB / DM-Verity

**Status** reads the current vbmeta flags without changing them. **Prepare AVB** optionally backs up the current vbmeta and boot images, then disables the current vbmeta verification/hashtree flags. It preserves other contents and Data encryption, performs full write readback, and attempts to restore the original bytes if the write fails.

An unlocked bootloader is required. **orange** indicates the unlocked state. Some firmware omits the other unlock properties; the tool accepts missing optional values but rejects explicit locked, unknown, or contradictory states.

Changing the flags makes the original vbmeta signature no longer match. With Back up first, the backup includes `vbmeta-original.img`, `vbmeta-disabled.img`, `boot.img`, checksums, and restore instructions. After changing boot/ROM contents, do not restore only the original vbmeta to re-enable verification. Return to verified stock using complete, matching stock firmware.

AVB verification and FBE encryption are separate. There is no need to run Disable FBE or clear Data just to disable AVB checks.

## Magisk

The recovery bundles the official Magisk **30.7** APK with its signed bytes preserved. A separate installer wrapper targets boot only, preventing old settings from redirecting installation to recovery/vendor_boot. Choose the backup option when running Magisk 30.7. Select writable storage first if you want a backup, or explicitly skip the backup. Reboot Android manually afterward and confirm the Magisk version and root access.

The user confirmed an upgrade to 30.7 through the Magisk App and a recovery installation after removing the previous installation. Installed Magisk can continue to be upgraded through the App.

Installer failure can leave Data changes; restoring boot is not a complete uninstall. If the boot image has no supported ramdisk, use the Magisk App image-patching method.

## Android SELinux

Check the actual state in Android after changing the setting. Recovery's SELinux state does not establish Android's state. The kernel, init, and modules can also affect the final mode. A stock user build may not accept permissive parameters.

**Restore Setting** uses the same backup storage to reverse this tool's parameter changes while retaining other current boot changes. After a kernel update, the tool rejects old records. SELinux changes and full boot restore still require AVB verification to be disabled.

## Install Image

**System / Vendor / Product / ODM Image** use the actual mappings of the current ROM. Select **Super** only for a complete `super.img`; the tool does not automatically resize partitions. After repartitioning, restart recovery to recreate mappings and refresh capacities.

The added targets check raw/Android sparse images against the current capacity. They reject read-only or still-mounted targets, a source file on the target partition, oversized images, and incomplete images. After writing, they sync and read back the specified data. Sparse `DONT_CARE` regions retain their existing contents. The original Boot / Recovery / Super entries use TWRP's existing handling.

**Persist, Optics, Prism, EFS, and Sec EFS** are physical partition targets. **EFS Image** writes `efs`; **Sec EFS Image** writes `sec_efs` and corresponds to Backup EFS. For Persist/EFS, use the matching raw image from the same phone. A normal TWRP file-backup archive cannot be flashed directly as an IMG.

Advanced Wipe hides the Metadata option but retains the partition and its existing handling. ROMs using metadata encryption may store encryption information there; do not wipe it casually.

## Confirmed behavior and remaining tests

The user confirmed stable recovery boot, pattern/PIN/password decryption on the tested custom-ROM configurations, Odin installation, Android Enforcing/Permissive, Keep TWRP persistence after reboot, and earlier Magisk installation.

Magisk 30.7 recovery reinstallation and the r2 optional-backup build have passed user device testing. Restore Setting, Boot Restore, module disable/restore, Mount RW, EFS backup, and the added image targets still need individual device tests. The previously tested lockscreen decryption components retain the same bytes in this release.
