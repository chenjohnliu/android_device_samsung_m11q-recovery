Unofficial recovery for **Samsung Galaxy M11 SM-M115F (m11q)**. Public revision **quokka r3**, built on 2026-10-24. Runtime version: `3.7.1_12-quokka-r3`.

## Downloads

- **TWRP-3.7.1_12-quokka-r3-m11q-20261024.tar** — Odin AP package containing one root-level `recovery.img`.
- **TWRP-3.7.1_12-quokka-r3-m11q-20261024.img** — the identical 64 MiB recovery image; install to **Recovery**.

GitHub asset digests identify the uploaded files. The TAR flashes only recovery; it contains no boot or vbmeta image.

## Changes since r2

- **GSI Tools** under Advanced > Essentials: Partition Status, separate swipe-confirmed **Delete Product**, and **Resize System** to the expanded size of a selected raw or Android sparse GSI. Resize never shrinks, implicitly deletes Product, resizes a filesystem or flashes the image.
- Before a layout change, save and read back a **Super metadata backup** to selected writable storage. Validate the non-A/B single-device linear layout and live unmounted mappings; attempt immediate metadata/mapping rollback on failure. Vendor and ODM contents are preserved.
- Replace the redundant Essentials Backup EFS shortcut with native **Backup > Sec EFS**, separate from **EFS**, using standard TWRP filesystem Backup/Restore.
- Group the unchanged Magisk 30.7 installer and **Root Modules** under **Essentials > Magisk**. No KernelSU/Next module-rescue support is added. Decrypted writable Data is required.
- Start the bundled Health HAL so fastbootd can initialize; use the tested Samsung fastboot USB identification. Fastbootd connection and logical size queries worked with the existing Samsung Windows driver.
- Complete eleven missing English resources, including `m11q_img_verifying`; audit both tracked and new sources for missing strings.
- Correct the GSI picker's style inheritance to prevent a null-font splash loop. Add a guard using the actual production style resolver and a negative fixture for the former invalid style.
- Preserve r2's optional backups for Prepare AVB, Keep TWRP and Magisk, existing guarded image writing, and the custom-ROM FBE decryption stack.

## Installation

Requires an already-unlocked bootloader. In Odin, select the TAR in **AP**, disable **Auto Reboot**, flash, then boot directly into TWRP after PASS.

### Stock ROM: keep TWRP after reboot

1. Boot directly into TWRP after flashing; complete these steps before starting Android.
2. In **Essentials > Select Storage**, choose writable microSD or USB OTG if you want backups. If no writable storage is available, explicitly choose **Skip backup** for each supported operation.
3. Check **Boot / AVB > AVB / DM-Verity > Status**. If verification is enabled, run **Prepare AVB**, choose **Back up first** or **Skip backup**, and swipe. Back up first saves current boot/vbmeta; both choices verify the flag update. If already disabled, no further preparation is needed.
4. Run **Essentials > Keep TWRP**, select the backup option and swipe. Confirm the log reports disabling the stock recovery restore files.
5. Reboot to System, then Recovery, and confirm TWRP remains installed.

Skip backup saves no persistent copy; in-memory originals support only immediate rollback during the operation. Keep TWRP renames restore files in place. This workflow does not require Disable FBE or Format Data. Magisk installation is optional and separate.

## Switching to a GSI

The user reports a successful switch from local **crDroid 9.20 (20261023)** to Andy's `lineage-20.0-20240215-UNOFFICIAL-gsi_arm64_gN.img` using the local predecessor with the same runtime feature sources. The formal r3 rebuild is separately packed/host-verified but has not been separately boot-tested. This does not establish compatibility for other ROM/GSI combinations.

1. Back up personal data and **Boot / Super** externally; keep a working full ROM ZIP. Boot the baseline ROM into Android once, then return to TWRP. Keep its kernel and SELinux setting.
2. Check **AVB Status**. The successful baseline had **vbmeta flags 3**, so no further Prepare AVB was needed.
3. For this unencrypted-Data workflow, run **Essentials > Disable FBE** on the compatible custom-ROM baseline while Vendor is present. Confirm success; stop on failure.
4. **GSI Tools > Delete Product**, then **Resize System** using the decompressed GSI. The tested image expands to **3378458624 bytes**. Both operations must succeed before continuing.
5. **Install > Install Image > GSI > System Image**. Wait for successful writing and readback verification.
6. **Wipe > Format Data > yes**. This erases internal storage. Disable FBE must have succeeded before this final format; formatting alone does not disable future encryption.
7. **Reboot > Recovery**, confirm normal startup, then **Reboot > System**.

Select writable backup storage before GSI Tools. Product deletion can prevent the old ROM from booting. `super-metadata.img` stores **only metadata**, not partition contents, and is not a flashable complete `super.img`. Old metadata cannot restore overwritten Product data. Do not proceed after any error.

## FBE and module support limits

**Stock ROM decryption and stock Enable/Disable FBE are unsupported.** Pattern/PIN/password decryption was confirmed only on tested custom-ROM configurations. Switching encryption mode changes configuration, does not decrypt existing files, and requires a separate manual Format Data. Prepare AVB preserves Data encryption and does not format it.

The GSI sequence above is a specific unencrypted-Data configuration; disabling FBE is not a universal GSI or normal-ROM requirement. No separate permissive kernel is required by that sequence.

On encrypted stock Data, module files can remain unreadable even when Data is mounted. An empty module list is not proof that no modules are installed. Root Modules retains the existing Magisk backend; it does not manage KernelSU/Next.

## Validation and sources

- Captured/synthetic liblp layouts cover capacity limits, independent deletion/growth, alignment, no shrink, invalid layouts and Vendor/ODM preservation; 13 transaction success/failure/rollback cases pass.
- Source and packed style/font checks pass; the injected invalid style is rejected. Strict English-resource audits find no missing/duplicate keys or unknown dynamic lookups.
- All 287 packed ELFs, FBE input pins, native library ownership, symbol closure, recovery AVB and TAR payload checks pass. Kernel, command line and runtime feature sources match the tested local predecessor. Only recovery/twrp and the ramdisk checksum list differ in the rebuilt ramdisk.
- Earlier user-confirmed FBE, Odin, fastbootd and r2 optional-backup results are retained. Native Sec EFS restore, module rescue and other individual image targets are not all device-tested.

See the [English guide](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/blob/fbe-reconstruction/ESSENTIALS.md), [繁體中文](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/blob/fbe-reconstruction/ESSENTIALS.zh-TW.md), and [exact build inputs](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/blob/fbe-reconstruction/BUILD_INFO-20261024-r3.json). Source changes are split into logical commits. The build record pins the exact device code revision; the release tag adds documentation only.

## Credits

- Thanks to @goldfish07 for the original TWRP device source and TeamWin for TWRP.
