# quokka Essentials 使用說明

適用 Samsung Galaxy M11（m11q）。從 Advanced → Essentials 進入。先用 Select Storage 選備份目的地，建議使用 Micro SD。備份存放於 `TWRP/Essentials/<裝置序號>/<操作與時間>/`，附校驗資訊。寫入操作需滑動確認；工具不會自動重開機或格式化。

| 選單 | 功能 |
| --- | --- |
| Disable FBE／Enable FBE | 切換 ROM 的資料加密設定。需要自行備份資料並手動 Format Data；與 recovery 解密既有 Data 的功能不同。 |
| Keep TWRP | 尋找 stock recovery 還原檔案，先備份再改名停用。修改 ROM 檔案前要求 hashtree 已停用。找不到檔案會明確回報；不解除 KG。 |
| Magisk 30.7 | 使用固定官方穩定版，先備份當前 boot，再安裝至有 ramdisk 的 boot。保留資料加密與 verity 設定。 |
| Android SELinux | 調整 Android boot 的 Enforcing／Permissive 參數；Restore Setting 撤回本工具的參數修改。每次保留 boot 備份。 |
| Mount RW | 在目前 recovery 工作階段掛載 system、vendor、product、odm 為可寫；確認實際結果，失敗不會假報成功。 |
| Root Modules | 列出或停用單個／全部模組，保存原狀態。Choose Module 選 module.prop；Restore States 選 modules.state。需要 Data 已解密可寫；不支援的 root 注入方式會拒絕操作。 |
| Boot / AVB | 備份／還原 boot，並提供 AVB / DM-Verity 子選單。還原前檢查格式、校驗值和 ROM fingerprint，再備份當前 boot。 |
| Backup EFS | 備份 sec_efs 原始映像。efs 是另一個分割區，此快捷工具不備份它。 |
| Diagnostics | 匯出 recovery 版本、SELinux、crypto 服務、boot 參數、掛載及日誌。 |

## AVB / DM-Verity

Status 唯讀顯示當前 vbmeta 驗證旗標。Prepare AVB 先備份當前 vbmeta 和 boot，再停用現有 vbmeta 的 verification／hashtree 旗標；保留其他內容和資料加密，寫入後完整讀回核對，失敗嘗試回復原內容。

需要 bootloader 已解鎖。orange 是解鎖狀態；部分 firmware 不提供其他解鎖屬性，工具接受缺省值，但拒絕明確鎖定、未知或矛盾的狀態。

旗標修改會使原始 vbmeta 簽章不再匹配。備份包含 `vbmeta-original.img`、`vbmeta-disabled.img`、`boot.img`、校驗值與還原說明。修改 boot／ROM 後，不應只還原原始 vbmeta 重新啟用驗證；回復 verified stock 應使用完整、匹配的原廠 firmware。

AVB 驗證與 FBE 加密是不同功能。沒有必要為了解除 AVB 檢查先 Disable FBE 或清空 Data。

## Magisk

新版整合固定官方 30.7 APK，保留原始簽署位元組；獨立安裝 wrapper 限定 boot 目標，避免舊設定導向 recovery／vendor_boot。先選 Micro SD 作備份儲存空間，再執行 Magisk 30.7。完成後自行重開 Android，確認 Magisk 版本及 root 權限。

使用者已確認從 Magisk App 升級至 30.7 可用，並完成移除後從本版本 recovery 重新安裝 30.7 的測試。已安裝的 Magisk可繼續從 App 升級。

安裝器失敗可能留下 Data 變更，還原 boot 不代表完整卸載。沒有 boot ramdisk 時，請使用 Magisk App 的映像修補方式。

## Android SELinux

切換後需在 Android 確認實際狀態。recovery 的 SELinux 狀態不能代替 Android 結果；kernel、init 和模組也可能影響最終模式。原廠 user build 未必接受 permissive 參數。

Restore Setting 使用相同備份儲存空間，撤回本工具的參數修改並保留其他當前 boot 變更。更新 kernel 後，工具會拒絕使用舊記錄。SELinux 修改與完整 boot 還原仍要求 AVB verification 已停用。

## Install Image

System／Vendor／Product／ODM Image 使用當前 ROM 的實際映射。完整 `super.img` 才選 Super；工具不自動調整分割區大小。重分割後需重新啟動 recovery，讓映射與容量重新建立。

新增入口依實際容量檢查 raw／Android sparse IMG；拒絕唯讀或仍掛載的目標、位於目標分割區上的來源、過大或不完整的映像。完成後同步並讀回核對指定資料；sparse DONT_CARE 區段保留原內容。原有 Boot／Recovery／Super 入口使用 TWRP 原本的處理方式。

Persist、Optics、Prism、EFS 和 Sec EFS 為實體分割區入口。EFS Image 寫入 `efs`；Sec EFS Image 寫入 `sec_efs`，與 Backup EFS 對應。Persist／EFS 請使用同一支手機的相應 raw 映像；一般 TWRP 檔案備份封裝不能直接當 IMG 刷入。

Advanced Wipe 隱藏 Metadata 選項，但保留其分割區與既有處理。採用 metadata encryption 的 ROM 可能在此保存加密相關資訊，不應任意清除。

## 已確認與待測

使用者已確認 recovery 穩定啟動、圖案／PIN／密碼解密、Odin 刷入、Android Enforcing／Permissive、Keep TWRP 重開機後保留，以及前版 Magisk 安裝。

Magisk 30.7 recovery 重新安裝已通過使用者實測。Restore Setting、Boot Restore、模組停用／還原、RW、EFS 備份與新增 Image 入口仍需逐項實測；此前通過的鎖屏解密元件在本版本保持相同位元組。
