Carrier Tweaks

一個 LSPosed 模塊，用來覆蓋電訊商設定（carrier config）：啟用 5G SA、VoNR、VoWiFi，並調整 5G 訊號閾值。

設定在每次系統讀取 carrier config 時即時套用，重啟後依然有效，不需要 Shizuku，也不需要每次開機手動再按。

功能

功能
修改的 carrier config
啟用 NR(5G) SA
carrier_nr_availabilities_int_array = [1, 2]（NSA + SA）
啟用 VoNR（Vo5G）
vonr_enabled_bool、vonr_setting_visibility_bool = true
啟用 VoWiFi
carrier_wfc_ims_available_bool、carrier_wfc_supports_wifi_only_bool、carrier_default_wfc_ims_roaming_enabled_bool、editable_wfc_mode_bool、editable_wfc_roaming_mode_bool = true
覆蓋 5G 訊號閾值
5g_nr_ssrsrp_thresholds_int_array = [-115, -105, -95, -85]

每個功能都可以在 App 內獨立開關，預設全部開啟。

需求

• 已 root，並安裝 LSPosed
• LSPosed 需要支援 libxposed API 102
• Android 12 或以上（minSdk 31）

使用方法

1. 安裝 APK
2. 在 LSPosed 管理器啟用模塊，作用域選 Phone Services（com.android.phone）
3. 從 LSPosed 管理器開啟本 App（直接從桌面開啟，可能連不上 LSPosed 服務）
4. 選擇需要的功能
5. 重啟手機（或強制停止 Phone Services）讓設定生效
6. 到系統設定確認 5G / VoNR / WiFi 通話選項是否出現

更改開關後，同樣需要重啟才會套用。

運作原理

模塊 hook com.android.phone 裡的 CarrierConfigLoader，在 getConfigForSubId* 回傳 carrier config 之前，把上表的設定寫入 PersistableBundle。系統其他部分（設定、SystemUI、電話服務）讀到的就是修改後的值。

設定開關透過 libxposed 的 RemotePreferences 在 App 與 hook 之間共用。

注意事項

• 這個模塊只改 Android 層的 carrier config。5G SA、VoNR、VoWiFi 是否真的可用，還取決於手機的數據機、韌體，以及電訊商網絡是否支援。
• 不包含 VoLTE 的 IMS provisioning（setImsProvisioningInt）。如果 VoLTE 本身未開，VoNR 與 VoWiFi 可能無法運作。
• getConfigForSubId* 的方法名稱可能因 Android 版本而異。如果沒有 hook 到，LSPosed 日誌會列出 CarrierConfigLoader 內回傳 PersistableBundle 的方法，方便修正。
• 修改系統電話服務有風險。如果通話異常，在 LSPosed 取消勾選本模塊並重啟即可還原。

自行編譯

需要 JDK 17。

gradle :app:assembleRelease

輸出在 app/build/outputs/apk/release/。

使用 GitHub Actions

repo 內已有 workflow（Actions → Build APK → Run workflow）。

要用自己的金鑰簽名，到 Settings → Secrets and variables → Actions 新增：

Secret
內容
KEYSTORE_BASE64
keystore 檔案的 base64
KEYSTORE_PASSWORD
keystore 密碼
KEY_ALIAS
金鑰別名
KEY_PASSWORD
金鑰密碼

沒有設定 secrets 時會用 debug 金鑰簽名。之後更新 APK 必須使用同一把金鑰，否則需要先解除安裝舊版。

依賴 libxposed API 102 需要 compileSdk 37（已在 gradle.properties 設定 android.suppressUnsupportedCompileSdk=37）。

致謝

• iKirby/PixelCarrierSettings：功能與設定值的參考來源
• libxposed
• LSPosed

授權

本專案以 GPL-3.0 授權。
