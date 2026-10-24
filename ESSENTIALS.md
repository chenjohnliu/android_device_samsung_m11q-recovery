# quokka Essentials Guide

English | [Traditional Chinese](ESSENTIALS.zh-TW.md)

For the Samsung Galaxy M11 (`m11q`). Open **Advanced > Essentials**. Backups use the destination chosen in **Select Storage**, such as Micro SD or USB OTG. They are saved under `TWRP/Essentials/<device-serial>/<operation-and-time>/` with verification information. Write operations require swipe confirmation. These tools do not automatically reboot or format the device.

Release [**quokka r3 (20261024)**](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/releases/tag/3.7.1_12-quokka-r3) adds GSI Tools, native Sec EFS backup and the Magisk submenu. It retains the backup choices tested in r2. **Prepare AVB**, **Keep TWRP**, and **Magisk** each offer **Back up first** or **Skip backup**. Skipping a backup still requires **Swipe without Backup**. If no writable storage is available, Skip backup works without SD or OTG.

Skip backup omits this tool's persistent backup. Device, bootloader, image, and write verification checks remain enabled. Original boot/vbmeta bytes held in memory can only support immediate rollback during the operation; there is no saved copy after reboot. Matching stock firmware may be needed for recovery. Keep TWRP retains the restore files by renaming them on their original partition rather than deleting them. Other backup, export, and restore tools still use the selected storage.

## Tools

| Menu | Function |
| --- | --- |
| Disable FBE / Enable FBE | **Not supported on stock ROM.** Changes the encryption configuration only on compatible custom ROMs. Back up your data and manually Format Data when switching modes. This does not decrypt existing data. |
| Keep TWRP | Finds stock recovery restore files, optionally backs them up, and disables them by renaming them. Hashtree verification must be disabled before ROM file changes. Reports when no supported files are found; it does not unlock KG. |
| Magisk 30.7 | Uses the bundled official stable version. Optionally backs up the current boot image, then installs into boot with a supported ramdisk. Preserves data encryption and verity settings. |
| Android SELinux | Changes Android boot parameters for Enforcing / Permissive. Restore Setting reverses the parameters changed by this tool. Each operation retains a boot backup. |
| Mount RW | Mounts system, vendor, product, and odm as writable for the current recovery session. Checks the actual result and reports failures. |
| Magisk > Root Modules | Retains existing Magisk module list, single/all disable and state restore. Requires decrypted writable Data. No KernelSU/Next support is added. |
| Boot / AVB | Backs up/restores boot and provides the AVB / DM-Verity submenu. Before restoring, validates the format, checksum, and ROM fingerprint, then backs up the current boot image. |
| GSI Tools | Reads allocation/free space, deletes Product with separate confirmation, or grows System to the expanded size of a selected raw/sparse GSI. Saves verified Super metadata before changes; never writes the GSI automatically. |
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

Installing a GSI does not by itself require disabling FBE. For a clean switch
between systems, follow the GSI's data-wipe instructions; formatting Data and
disabling future encryption are different actions. Keep FBE if the final
kernel/vendor/GSI combination supports it. The M11's `fileencryption=ice` format
is vendor-specific: successful TWRP decryption does not prove that a particular
GSI's Android userspace supports the same configuration. Follow that GSI's
documented requirements or diagnose its boot logs before changing encryption.
See [AOSP FBE](https://source.android.com/docs/security/features/encryption/file-based)
and [GSI installation guidance](https://source.android.com/docs/core/tests/vts/gsi).

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

## Fastbootd

The **quokka-fastboot1** source update starts the bundled Health 2.1 service and
uses Samsung USB identification for fastboot. This fixes startup waiting for
Health and allows the tested Windows host to use its existing Samsung Android
Bootloader driver. An additional Google USB driver was not required in this test.

From TWRP, use these commands on the connected computer to enter fastbootd and
check the connection and current layout:

```sh
adb reboot fastboot
fastboot devices
fastboot getvar is-userspace
fastboot getvar is-logical:system
fastboot getvar partition-size:system
fastboot getvar partition-size:product
```

`is-userspace` must report `yes`. The tested non-A/B layout uses `system` and
`product` without `_a`/`_b` suffixes. Return to Recovery with:

```sh
fastboot reboot recovery
```

These commands only query partitions. Successful queries do not validate a GSI's
compatibility, deletion, resizing or flashing. The Install Image targets still
use the current partition capacity and do not resize automatically. Format Data
does not enlarge System. See [fastboot1 build notes](RELEASE_NOTES-20261024-fastboot1.md)
for exact tested images, source revisions and validation scope.

## GSI Tools

[**quokka r3**](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/releases/tag/3.7.1_12-quokka-r3) replaces the redundant Essentials EFS shortcut with this submenu.
The user reports a successful Andy GSI switch and boot from a crDroid baseline
using the local predecessor with the same runtime feature sources. The formally
named rebuild has not been separately boot-tested; other ROM/GSI combinations
are not established by that result.

- **Partition Status** reads slot-zero partition allocations, group limits and
  free allocatable Super space. Group limits can restrict growth independently.
- **Delete Product** is a separate swipe-confirmed operation. It removes the
  Product logical partition and mapping, releasing its allocation. This can
  prevent the current ROM from booting. Back up the complete ROM first.
- **Resize System** selects a raw or Android sparse `.img`, validates it, and
  grows System allocation to its expanded size, rounded to the logical block
  boundary. It never shrinks System, deletes Product, resizes the filesystem,
  or writes image data. If space is insufficient, it stops.

Choose writable backup storage first. Before each actual layout change, the
tool saves and reads back `super-metadata.img` under `TWRP/Essentials/.../gsi-*`.
This is **metadata only**, with no System/Product contents. It is not a complete
`super.img` and must not be flashed through Install Image. The tool attempts
immediate metadata/mapping rollback on failure. Once another image reuses freed
extents, restoring old metadata cannot restore overwritten Product contents.

Only the non-A/B m11q single-device linear layout is supported. Snapshot,
slotted, multi-device, mounted or stale target mappings are rejected. Vendor,
ODM, boot, recovery, vbmeta and Data are not changed. After preparation, use
**Install Image > System Image** separately. Successful allocation does not
establish GSI boot compatibility or fix FBE/SELinux/APEX problems.

## Confirmed GSI switch sequence

The reported working combination used local crDroid 9.20 (20261023) and Andy's
`lineage-20.0-20240215-UNOFFICIAL-gsi_arm64_gN.img`. This specific workflow uses
unencrypted Data; disabling FBE is not a universal requirement for normal ROM
use or every GSI.

1. Back up personal files and Boot/Super to SD/OTG; retain a working full ROM
   ZIP. Boot the baseline into Android once, then return to TWRP. Keep its kernel
   and SELinux setting.
2. Check AVB Status. The successful baseline already had vbmeta flags 3; no
   further Prepare AVB was needed in that state.
3. Run Essentials > Disable FBE on the compatible custom ROM and confirm
   success. Stop if it fails.
4. Run GSI Tools > Delete Product, then Resize System using the decompressed
   GSI. The example expands to 3378458624 bytes; never use compressed file size.
5. Install > Install Image > GSI > System Image; confirm successful writing and
   readback verification.
6. Wipe > Format Data > yes. This erases internal storage. Disable FBE must have
   succeeded before this final format.
7. Reboot > Recovery, confirm TWRP starts, then Reboot > System.

No separate permissive kernel is part of this sequence. Other ROM/GSI
combinations still require compatibility validation.

## Root Modules

Open **Essentials > Magisk > Root Modules**. The existing Magisk module list,
single/all disable and saved-state restore behavior is retained. KernelSU and
KernelSU Next module rescue are not supported by this release. Data must be decrypted and
writable. Select `module.prop` to disable one module, or `modules.state` to
restore saved states. The existing rejection of unsupported KernelSU initrc
injection remains. Device testing of module rescue is still pending.

## Install Image

**System / Vendor / Product / ODM Image** use the actual mappings of the current ROM. Select **Super** only for a complete `super.img`; the tool does not automatically resize partitions. After repartitioning, restart recovery to recreate mappings and refresh capacities.

The added targets check raw/Android sparse images against the current capacity. They reject read-only or still-mounted targets, a source file on the target partition, oversized images, and incomplete images. After writing, they sync and read back the specified data. Sparse `DONT_CARE` regions retain their existing contents. The original Boot / Recovery / Super entries use TWRP's existing handling.

**Persist, Optics, Prism, EFS, and Sec EFS** are physical partition targets. **EFS Image** writes `efs`; **Sec EFS Image** writes `sec_efs`. Native **Backup > Sec EFS** now backs up its existing filesystem through standard TWRP Backup/Restore; **EFS** remains a distinct backup. Existing filesystem mounts are preserved. For Persist/EFS, use the matching raw image from the same phone. A normal TWRP file-backup archive cannot be flashed directly as an IMG.

Advanced Wipe hides the Metadata option but retains the partition and its existing handling. ROMs using metadata encryption may store encryption information there; do not wipe it casually.

## Confirmed behavior and remaining tests

The user confirmed stable recovery boot, pattern/PIN/password decryption on the tested custom-ROM configurations, Odin installation, Android Enforcing/Permissive, Keep TWRP persistence after reboot, and earlier Magisk installation.

Magisk 30.7 recovery reinstallation and the r2 optional-backup build have passed user device testing. Restore Setting, Boot Restore, module disable/restore, Mount RW, EFS backup, and other image targets still need individual device tests. The previously tested lockscreen decryption components retain the same bytes in this release.
