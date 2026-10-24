# strings1 resource update — 2026-10-24

Adds eleven missing English keys: nine m11q_img_* messages plus restore and
zip_compatible_err. Existing definitions and translations are preserved. Other
languages continue to inherit the English base. Ten added values come from
existing caller defaults; Restore matches the existing button and ORS operation.

tools/check_string_resources.py audits message/lookup APIs, partition-name
forwarding, adjacent C++ literals, built-in theme XML and both resource-reference
syntaxes. Reviewed dynamic forwarding layers are recorded; unknown dynamic calls
fail strict checking. Source and packed checks cover 889 source/header files,
13 theme XMLs and 736 referenced keys, with zero missing or duplicate definitions.
A real-key deletion negative test fails for m11q_img_verifying as expected.

From the Android source root:

```sh
python3 bootable/recovery/tools/check_string_resources.py \
  --root bootable/recovery --strict --report /tmp/quokka-string-audit.json
```

Add --language-file /path/to/extracted/twres/languages/en.xml to check the
language file extracted from a generated recovery image against the same source.

## Provenance and validation

- Recovery source: [cc9238475f5e](https://github.com/chenjohnliu/android_bootable_recovery/commit/cc9238475f5e3a2afcb1fae497106228a6d1c9c9).
- Device source: [15f43e91b059](https://github.com/chenjohnliu/android_device_samsung_m11q-recovery/commit/15f43e91b059785d58306cbe570c2bef582c37ae); unchanged code and FBE input pins.
- QCOM common: 70ecbe3018bf600de7d4cc452b66a543909d0673.
- System core: 9919f11d0c00885006d2ca22ef034ae8eaf5bff0.

All FBE input, ELF integrity, native-owner, tool/version and AVB checks pass.
Kernel and command line match fastboot1. Of 287 ELFs, 286 are identical;
recovery is rebuilt with the current source revision marker. No runtime C++,
image-writing, USB, Health, partition or encryption behavior changes.

| Local artifact | SHA-256 |
| --- | --- |
| TWRP-3.7.1_12-quokka-fastboot1-strings1-m11q-20261024.img | fc15957cf956ea8b68b44fba9aeba920c2b7beae060f84a4301c03f9de7ab581 |
| Matching Odin TAR | 62aaf8d751498f3a1fbbd063b69fcbe75399a0e7799b120604e03c0596e017e7 |

The 64 MiB image matches the single recovery.img in the TAR. The UI version
remains quokka-fastboot1; strings1 is the artifact suffix. These hashes identify
the checked local build. This source push does not publish new release assets.
This image's boot/runtime test remains pending; no phone operations were made
for this resource update. Historical fastboot1 validation and release notes remain.
