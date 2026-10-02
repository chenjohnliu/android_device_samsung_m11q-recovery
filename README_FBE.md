# Samsung Galaxy M11 TWRP FBE rebuild

This branch reconstructs TWRP 12.1 with a consistent Android 13 crDroid
crypto stack for m11q / SM-M115F using `fileencryption=ice`.

## Required companion commits

| Checkout | Fork branch | Commits |
| --- | --- | --- |
| device/qcom/twrp-common | chenjohnliu/android_device_qcom_twrp-common: m11q-32bit-services | a2b52d9a3789851b62dd86d879f4e5365c14d586, 70ecbe3018bf600de7d4cc452b66a543909d0673 |
| system/core | chenjohnliu/android_system_core: m11q-fbe-diagnostics | 9919f11d0c00885006d2ca22ef034ae8eaf5bff0 |
| bootable/recovery | chenjohnliu/android_bootable_recovery: m11q-fbe-essentials | 02f9022bfbacba6db4b73eacd62061e93b87b50a |

The common init commits select ELF32 library paths for the bundled vendor
services. The system/core commit builds a separate tombstoned diagnostic
module; normal Android tombstoned keeps its original paths. The recovery
commit adds Advanced > Essentials and requires the two addon ZIPs here.

## Inputs and rebuilding

`manifests/twrp12.1-base.xml` preserves all original source revisions before
these commits. Override its four checkouts with the branches above and this
device branch, or cherry-pick the commits onto those exact base revisions.
Do not run repo sync against the base manifest after selecting the forks.

The checked-in `FBE_INPUTS.sha256` records every staged ramdisk file and
kernel. Check them with `sha256sum -c FBE_INPUTS.sha256` from this device
directory. The matching ROM input revisions are:

- crDroid device: 97206c0e000228a92b690edbd0f8d14e6147cd00
- crDroid vendor: 6a8923918e90b734913cab84bc8ac7022a2dc143
- Native keystore XML: system/security 14737db1429b8eebc15568bc748b2cd79ccad5c2

From the Android checkout:

```sh
export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh
lunch twrp_m11q-eng
mka -j4 recoveryimage
```

The packed ramdisk must contain the complete ELF32 linker/runtime and QSEE
dlopen set, compatible VINTF declarations, Gatekeeper implementation,
debuggerd init sockets, and `prepdecrypt.setpatch=true` before flashing.
The latter uses readonly mounts to read the installed ROM's actual OS and
system/vendor security patch levels before Keymaster starts.

## Evidence and limits

A complete recovery flash/readback and fresh recovery reboot validated
systemwide/DE/CE decryption and readable internal storage on the current
crDroid 13 installation with default/no-lockscreen authentication.
PIN/password/pattern configurations are not yet tested. The historical
qseecomd SIGSEGV did not recur; its original root cause is unknown.

The FBE image tested on the phone has SHA-256
`d4ba6989166189c5866c320e4996a53d438b739d1a1642aa3eaf77ba18e5f380`.
The subsequent Essentials image has SHA-256
`71ce31df1ccf741d5b1f1f8c98eed8a58bd761418c1efd34ef0701a44192e770`.
Its packed files, ELF hashes, GUI routes, shell syntax and AVB verification
passed; it has not been flashed, and its new menu is not yet runtime tested.

## Essentials payloads

Source release: https://github.com/chenjohnliu/android_device_samsung_m11q/releases/tag/20260926-13-Cherish

| Menu | Bundled path | Original asset | SHA-256 |
| --- | --- | --- | --- |
| Disable FBE | /addon/decrypt.zip | Disable_FBE.zip | 3e3587f9dd1ab258f8e0941b7cfe3b78b4e623a53fac21e512c7d462362430ee |
| Enable FBE | /addon/encrypt.zip | Enable_FBE.zip | 291663361964d1fe19c2c8b97ba87dc7e708c07a6caf5401ee1b13d0637fda02 |

These exact release ZIPs switch the vendor fstab's encryption configuration.
They do not unlock existing encrypted files or automatically format data.
Changing mode requires manual **Format Data**, which erases internal
storage, followed by reboot to Recovery before System. A factory reset is
insufficient. Do not execute these switches as a test of real FBE unlock.
Only these two Essentials entries were restored from the old recovery.
