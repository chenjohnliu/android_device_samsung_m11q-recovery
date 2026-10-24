# quokka-fastboot1 build notes — 2026-10-24

## Changes

- Start the existing bundled Health 2.1 HAL at boot. Without it, fastbootd waits
  for IHealth/default before it can create USB endpoints.
- Use Samsung USB 04E8:6860 in fastboot mode. Restore the previous 18D1 ADB,
  sideload and MTP identities when switching modes. The tested host uses its
  existing Samsung Android Bootloader driver without an additional Google driver.
- Add the missing m11q_img_capacity English resource while retaining the existing
  capacity check and error text. Images larger than System remain rejected.

## Validation

Verified on SM-M115F: Recovery image write/readback, Recovery boot, automatic
Health startup, entry through the real fastbootd boot path, is-userspace=yes,
logical System/Product queries, all four partition sizes, and return to Recovery
with ADB/MTP working. No Product deletion, resizing, GSI installation or Data
formatting was performed. Query verification does not establish GSI compatibility.

| Partition | Bytes in the tested layout |
| --- | ---: |
| system | 2516582400 |
| product | 691011584 |
| vendor | 691011584 |
| odm | 1323008 |

Packed FBE inputs, native library ownership, all 287 ELF files, existing tools,
version and AVB checks pass. Kernel and command line match msgfix. Of 287 ELFs,
285 are byte-identical; recovery/twrp are rebuilt for the candidate version.
The FBE input list refreshes only the two intentionally changed init RC hashes.
Existing stock FBE limitations and other tools' individual test limits remain.

## Exact source revisions

- bootable/recovery: [c31a88d333ab](https://github.com/chenjohnliu/android_bootable_recovery/commit/c31a88d333ab80aa25775c165f0e604d0b00aab5)
- device/samsung/m11q: [d9b5f49dfdc4](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/commit/d9b5f49dfdc4f37d9e48e95abe6effe01895cecc) (code revision before this documentation update)
- device/qcom/twrp-common: 70ecbe3018bf600de7d4cc452b66a543909d0673
- system/core: 9919f11d0c00885006d2ca22ef034ae8eaf5bff0

Use manifests/twrp12.1-base.xml for the remaining base revisions and override the
four projects above. FBE_INPUTS.sha256 records the exact kernel and staged blobs.

## Tested local artifacts

These names and hashes identify the tested build; this source update does not
publish a new GitHub release or replace the existing r2 download assets.

| Artifact | SHA-256 |
| --- | --- |
| TWRP-3.7.1_12-quokka-fastboot1-m11q-20261024.img | 888d7ecfe6f0d13fced7c80d71638af4f525e18178901f018e589be566042765 |
| TWRP-3.7.1_12-quokka-fastboot1-m11q-20261024.tar | ae72e3a3e9305a076b890c60918a0d40587263c39c98651ca891330c8e254add |

The 64 MiB IMG and the single root recovery.img in the Odin TAR are identical.
GSI Tools, Sec EFS backup integration and KernelSU-family module changes are
separate work and are not included.
