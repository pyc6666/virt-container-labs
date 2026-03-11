# VMware Workstation Pro 安裝與入門操作手冊（W01 補充）

## 1　文件說明

本手冊是 W01 課堂的實作補充教材。

### 三個成功訊號

完成本手冊後，你應該能做到：

1. 開啟 VMware Workstation Pro 主視窗
2. 建立並啟動一台 VM
3. 完成一次 Snapshot 建立與回復

在任何步驟卡住時，先對照這三個訊號判斷自己卡在哪個階段，再到第 7 節查對應的排錯方法。

---

## 2　安裝前檢核

進入安裝前，逐項確認以下條件。建議每組用一張勾選表簽核，避免裝完才回頭補環境。

| # | 項目       | 要求                                                 | 如何確認                                                             |
| :-: | ---------- | ---------------------------------------------------- | -------------------------------------------------------------------- |
| 1 | CPU        | 64-bit x86 / AMD64，2011 年後相容處理器              | `設定 > 系統 > 關於` 查看處理器型號                                |
| 2 | 硬體虛擬化 | Intel VT-x 或 AMD-V，BIOS / UEFI 已啟用              | 工作管理員 > 效能 > CPU，查看「虛擬化」欄位是否顯示「已啟用」        |
| 3 | Host OS    | 在官方支援清單內（參見 VMware KB80807）              | `winver` 指令確認 Windows 版本與組建編號                           |
| 4 | RAM        | 最低 2 GB，建議 4 GB 以上                            | `設定 > 系統 > 關於` 查看已安裝的 RAM                              |
| 5 | 磁碟       | 安裝約需 2.5 GB，另預留 guest 空間（建議至少 40 GB） | 檔案總管查看目標磁碟的可用空間                                       |
| 6 | 顯示卡     | 16-bit 或 32-bit，建議最新驅動                       | `裝置管理員 > 顯示卡介面` 確認驅動版本                             |
| 7 | 網路       | 主機可用 Ethernet 控制器                             | `裝置管理員 > 網路介面卡` 確認有可用網卡                           |
| 8 | 帳號權限   | 本機管理員（local Administrators 群組成員）          | 命令提示字元執行 `net localgroup Administrators`，確認目前帳號在列 |
| 9 | 相容性     | 沒有不相容的 VMware 產品殘留                         | `控制台 > 程式和功能` 檢查是否有舊版 VMware 產品未移除             |

### 2.1　關於硬體虛擬化（VT-x / AMD-V）

硬體虛擬化是執行 64-bit guest OS 的必要條件。如果工作管理員顯示「虛擬化：已停用」，需要進入 BIOS / UEFI 手動啟用：

- **Intel 主機**：在 BIOS 中找到 `Intel Virtualization Technology`（有時縮寫為 VT-x 或 Intel VT），設為 Enabled。
- **AMD 主機**：在 BIOS 中找到 `SVM Mode`（Secure Virtual Machine），設為 Enabled。
- 設定完成後儲存並重新開機，回到工作管理員確認狀態已變更。

不同廠牌主機板進入 BIOS 的按鍵不同（常見：Del、F2、F10、F12），開機時注意畫面提示或查閱主機板說明書。

### 2.2　關於 Hyper-V 共存

Windows 10 / 11 的部分功能（如 WSL2、Windows Sandbox、Credential Guard）會啟用 Hyper-V。VMware Workstation Pro 自 15.5.5 版起支援與 Hyper-V 共存，但效能會有差異。如果安裝後遇到 VM 執行異常緩慢，可先確認是否有 Hyper-V 相關功能開啟：

```powershell
# 查看 Hyper-V 狀態
systeminfo | findstr /i "hyper-v"
```

若不需要 Hyper-V，可在「Windows 功能」中關閉，或在進階啟動選項中停用：

```powershell
bcdedit /set hypervisorlaunchtype off
```

關閉後需重新開機才會生效。

---

## 3　安裝與首次啟動（Windows）

以下流程約 15–20 分鐘。

### 3.1　取得安裝檔

VMware Workstation Pro 目前為免費授權模式，不需購買 license key。安裝檔可從 Broadcom 支援入口網站下載：

1. 前往 Broadcom Support Portal 並登入（需註冊免費帳號）。
2. 搜尋 VMware Workstation Pro，選擇最新版本下載。
3. 下載完成後確認檔名格式為 `VMware-workstation-full-xx.x.x-xxxxxxx.exe`，檔案大小約 600–700 MB。

> 課堂建議：由助教預先下載好安裝檔放在共享資料夾或隨身碟，避免全班同時下載塞爆網路。

### 3.2　安裝主程式

1. **以本機管理員身份登入 Windows**。非管理員帳號會導致安裝精靈無法寫入系統目錄或註冊服務。
2. **關閉可能干擾安裝的程式**：防毒軟體的即時防護、其他虛擬化軟體（VirtualBox、舊版 VMware）、佔用大量磁碟 I/O 的程式。
3. **雙擊安裝檔**，進入安裝精靈。
   - 若雙擊無反應或彈出權限錯誤：右鍵 → 以系統管理員身分執行。
   - 若 Windows SmartScreen 跳出警告：點選「其他資訊」→「仍要執行」。
4. **安裝精靈畫面說明**：
   - **Welcome 頁面**：確認版本號，點選 Next。
   - **EULA（使用者授權合約）**：勾選 I accept the terms，點選 Next。
   - **Custom Setup**：建議使用預設安裝路徑（`C:\Program Files (x86)\VMware\VMware Workstation\`）。不建議變更預設目錄，因為可能導致部分功能路徑錯誤或安全性問題。此頁面也可選擇是否安裝增強型鍵盤驅動（Enhanced Keyboard Driver），建議勾選。
   - **User Experience Settings**：可選擇是否加入 VMware 客戶體驗改善計畫（CEIP）及啟動時自動檢查更新。課堂環境建議兩者都取消勾選，避免不必要的網路流量。
   - **Shortcuts**：勾選桌面捷徑與 Start Menu 資料夾，方便後續啟動。
   - **Ready to Install**：確認設定摘要，點選 Install 開始安裝。
5. **安裝過程**約需 3–5 分鐘，期間會安裝虛擬網路介面卡（vmnet）和虛擬 USB 裝置驅動。
6. **安裝完成後若精靈要求重開機，立即重開**。跳過重開機可能導致虛擬網路服務無法正常啟動。

### 3.3　安裝後驗證

安裝完成並重開機後，確認以下三件事：

1. **VMware 服務已啟動**：按 `Win + R` 輸入 `services.msc`，找到以下服務確認狀態為「執行中」：

   - `VMware Authorization Service`（vmauthdService）
   - `VMware Workstation Server`（vmware-wssd）
   - `VMware NAT Service`（VMwareHostd 的子服務）
   - `VMware DHCP Service`
2. **虛擬網路介面卡已建立**：`裝置管理員 > 網路介面卡` 中應出現以下虛擬網卡：

   - `VMware Virtual Ethernet Adapter for VMnet1`（Host-only）
   - `VMware Virtual Ethernet Adapter for VMnet8`（NAT）
3. **遠端連線與 VM 共享預設為啟用** — 這是 VMware 的預設行為，不是故障。若課堂環境不需要這些功能，可在 `Edit > Preferences > Shared VMs` 中關閉。

### 3.4　首次啟動

1. 從 `Start > Programs > VMware > VMware Workstation` 啟動（或雙擊桌面捷徑）。
2. 首次啟動會出現 EULA 確認畫面，同意後進入主視窗。
3. **看到主視窗（顯示 Home 標籤、Library 面板、選單列）→ 成功訊號 ① 達成。**

主視窗的重要區域：

- **Home 標籤**：快速建立 VM、開啟 VM、連線到遠端伺服器。
- **Library 面板**（左側）：列出所有已建立的 VM，可快速切換。
- **選單列**：`File`（建立/開啟 VM）、`Edit > Preferences`（全域設定）、`Edit > Virtual Network Editor`（網路設定）。

---

## 4　建立第一台 VM

目的：驗證安裝「不只打得開，而是可用」。

### 4.1　建立精靈流程

1. 在主視窗點選 `Create a New Virtual Machine`（或 `File > New Virtual Machine`）。
2. **精靈類型選擇**：
   - **Typical（推薦）**：自動配置大部分硬體設定，適合快速建立。W01 課堂使用 Typical 即可。
   - **Custom（進階）**：可自訂硬體相容性版本、記憶體、處理器核心數等，後續課程再使用。
3. **安裝來源**：
   - 若有 OS 安裝 ISO：選 `Installer disc image file (iso)` 並指定路徑。VMware 會自動偵測 OS 類型並啟用 Easy Install（自動安裝）。
   - 若暫時沒有 ISO：選 `I will install the operating system later`，先建立空 VM 再掛載 ISO。
4. **Guest OS 類型**：依安裝的 OS 選擇（例如 Microsoft Windows → Windows 11 x64）。這會影響 VMware 預設分配的硬體資源。
5. **VM 名稱與儲存位置**：
   - 名稱建議用有意義的命名（例如 `W01-Ubuntu-Test`），避免使用預設的 `Windows 11 x64`。
   - 儲存位置建議放在 SSD 上以獲得較好的磁碟效能。預設路徑為 `C:\Users\<使用者>\Documents\Virtual Machines\`。
6. **磁碟大小**：
   - Typical 模式會根據 Guest OS 類型給出建議值（Windows 11 建議 60 GB，Ubuntu 建議 20 GB）。
   - 選擇 `Store virtual disk as a single file`（效能較好）或 `Split virtual disk into multiple files`（方便搬移到 FAT32 磁碟）。
   - 此處設定的是最大容量，實際佔用空間會隨使用動態增長（thin provisioning）。
7. **確認摘要 → Finish**。VM 建立完成，出現在 Library 面板中。

### 4.2　硬體設定調整（建立後）

建立完成後，可在 VM 的 `Edit virtual machine settings` 中調整硬體：

| 項目       | 建議最小值 | 課堂建議值 | 說明                                           |
| ---------- | ---------- | ---------- | ---------------------------------------------- |
| Memory     | 2 GB       | 4 GB       | 低於 2 GB 可能導致 guest OS 安裝失敗或極度緩慢 |
| Processors | 1 核心     | 2 核心     | 不要超過 host 實體核心數的一半                 |
| Hard Disk  | 20 GB      | 60 GB      | thin provisioning，實際佔用隨使用增長          |
| Network    | NAT        | NAT        | 教室通用預設，詳見第 5 節                      |
| Display    | 自動       | 自動       | 勾選 Accelerate 3D graphics 可改善圖形效能     |

### 4.3　啟動 VM

1. 在 Library 中選擇剛建立的 VM。
2. 點選 `Power on this virtual machine`（綠色播放按鈕）。
3. 若有掛載 ISO，VM 會從 ISO 開機進入 OS 安裝流程。
4. 若選擇「稍後安裝 OS」，VM 會開機到 BIOS / UEFI 畫面或顯示 `Operating System not found`，這是正常行為。
5. **VM 成功 Power On 並進入開機畫面 → 成功訊號 ② 達成。**

> **VMware Tools**：OS 安裝完成後，強烈建議安裝 VMware Tools（`VM > Install VMware Tools`）。它提供更好的滑鼠整合、顯示卡驅動、共用資料夾、時間同步等功能。W01 課堂不強制要求，但後續課程會用到。

---

## 5　網路模式：NAT / Bridged / Host-only

VMware Workstation 安裝時會自動建立多個虛擬網路介面卡（vmnet），每個 VM 可獨立指定使用哪種網路模式。理解這三種模式的差異是虛擬化基礎中最重要的概念之一。

### 5.1　模式對照表

| 模式                | 拓樸說明                                                                                                | 典型使用情境                                      | IP 配置方式                                                 | 對外連線能力                  | 風險提示                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------------- | ----------------------------- | ------------------------------------------------------------ |
| **NAT**       | VM 透過 host 上的虛擬 NAT 設備存取外網；host 負責位址轉換，VM 的流量對外網來說看起來都是 host 發出的    | 教室通用預設，學生機器快速上網、下載套件          | VMware 內建 DHCP 自動配發私有 IP（預設網段 192.168.x.0/24） | VM → 外網 ✓；外網 → VM ✗  | 受主機 NAT / 防火牆策略影響                                  |
| **Bridged**   | VM 直接橋接到 host 所在的實體網段，取得與 host 同級的網路地位，就像一台獨立的實體機器插在同一個交換器上 | VM 需要與實體網段內其他機器直接互連、架設對內服務 | 由實體網段的 DHCP 伺服器配發 IP（與 host 同網段）           | VM → 外網 ✓；外網 → VM ✓* | 受校園 / 企業網路政策與防火牆影響；某些網路環境限制 MAC 數量 |
| **Host-only** | VM 只與 host 及其他掛在同一 Host-only 網段的 VM 互連，完全不碰實體網路                                  | 內部實驗網、隔離測試、模擬封閉環境                | VMware 內建 DHCP 配發私有 IP                                | 預設無外網連線                | 完全隔離，適合安全性實驗                                     |

\* Bridged 模式下外網能否連入取決於實體網路防火牆策略。

### 5.2　連線能力矩陣

| 連線方向                | NAT | Bridged | Host-only |
| ----------------------- | :--: | :-----: | :-------: |
| VM → 外網              |  ✓  |   ✓   |    ✗    |
| 外網 → VM              | ✗† |   ✓   |    ✗    |
| VM ↔ Host              |  ✓  |   ✓   |    ✓    |
| VM ↔ 同網段其他 VM     |  ✓  |   ✓   |    ✓    |
| VM ↔ 實體 LAN 其他機器 |  ✗  |   ✓   |    ✗    |

† NAT 模式可透過 host 設定 port forwarding 允許外網連入（`Edit > Virtual Network Editor > NAT Settings > Port Forwarding`）。

### 5.3　選擇決策流程

```
VM 需要存取外網嗎？
├─ 否 → Host-only
└─ 是 → 實體網段的其他機器需要直接存取此 VM 嗎？
         ├─ 是 → Bridged
         └─ 否 → 需要從外網主動連入 VM 嗎？
                  ├─ 是 → Bridged 或 NAT + port forwarding
                  └─ 否 → NAT（教室建議預設）
```

### 5.4　如何切換網路模式

1. 選擇目標 VM → `VM > Settings`（或右鍵 → Settings）。
2. 在 Hardware 標籤中點選 `Network Adapter`。
3. 右側面板可選擇：
   - `Bridged: Connected directly to the physical network`
   - `NAT: Used to share the host's IP address`
   - `Host-only: A private network shared with the host`
   - `Custom: Specific virtual network`（可指定特定 vmnet）
4. 切換後點選 OK。若 VM 正在執行，新設定會立即生效，guest OS 內需要重新取得 IP（`ipconfig /release` + `ipconfig /renew` 或重新啟動網路介面）。

### 5.5　Virtual Network Editor

`Edit > Virtual Network Editor`（需要管理員權限，點選左下角 `Change Settings`）可以進行進階網路設定：

- 修改各 vmnet 的子網段與子網路遮罩。
- 設定 NAT 的 port forwarding 規則。
- 設定 DHCP 的 IP 配發範圍。
- 新增 / 移除自定義虛擬網路。
- 還原所有網路設定為預設值（`Restore Defaults`，會刪除所有自訂設定）。

> W01 課堂不需要修改 Virtual Network Editor 的設定，使用預設即可。此處先了解入口位置，後續課程會深入操作。

---

## 6　Snapshot 建立與回復

### 6.1　概念

Snapshot 保存 VM 在某個時間點的完整狀態，包括記憶體內容、磁碟狀態、硬體設定。建立 Snapshot 後，無論對 VM 做了什麼變更（安裝軟體、修改設定、搞壞系統），都可以瞬間回到 Snapshot 當時的狀態。

Snapshot 是樹狀結構，可以從同一個基準點分岔出多個分支：

```
[Snapshot 1: "Clean OS Install"]
    ↓
  安裝 Web Server + Database
    ↓
[Snapshot 2: "Dev Environment Ready"]
    ├─ 測試配置 A → [Snapshot 3a: "Config A"]
    └─ 測試配置 B → [Snapshot 3b: "Config B"]
            ↓
        部署失敗 → ⤴ Revert to Snapshot 2
```

**Snapshot 不是備份**：Snapshot 依賴原始磁碟檔案，如果原始 `.vmdk` 損毀，Snapshot 也無法使用。長期保存請用完整的 VM 複製或匯出 OVF。

### 6.2　操作流程（5 步驟）

| 步驟 | 操作                                           | 預期結果                         |
| :--: | ---------------------------------------------- | -------------------------------- |
|  1  | 在 VM 穩定狀態下建立 Snapshot                  | Snapshot 出現在 Snapshot Manager |
|  2  | 做一個可觀測變更（修改設定檔、安裝套件等）     | 可明確觀測到狀態改變             |
|  3  | Revert 到 Snapshot                             | VM 回到步驟 1 的狀態             |
|  4  | 驗證狀態確實恢復（步驟 2 的變更已消失）        | 與步驟 1 一致                    |
|  5  | 提交「建立 → 變更 → 回復 → 驗證」完整證據鏈 | TA 可在 10 分鐘內複核            |

完成步驟 4 → **成功訊號 ③** 達成。

### 6.3　詳細操作說明

**建立 Snapshot**：

1. 確保 VM 處於穩定狀態（已開機且無正在執行的安裝程序，或已正常關機）。
2. 選單 `VM > Snapshot > Take Snapshot`（或按 `Ctrl + Shift + S`）。
3. 輸入 Snapshot 名稱（例如 `Clean Install`）與描述（例如「作業系統安裝完成、尚未安裝任何額外軟體」）。
4. 點選 `Take Snapshot`。完成後在 Snapshot Manager 中可看到新建立的節點。

**查看 Snapshot Manager**：

- `VM > Snapshot > Snapshot Manager`（或按 `Ctrl + Shift + M`）。
- 畫面會顯示 Snapshot 樹狀結構，標示「You Are Here」指示 VM 目前所在的狀態。

**回復 Snapshot（Revert）**：

1. `VM > Snapshot > Revert to Snapshot`（回到最近一次的 Snapshot）。
2. 或在 Snapshot Manager 中選擇任意 Snapshot → 點選 `Go To`。
3. 系統會彈出確認對話框，提醒目前的狀態變更將會遺失。確認後 VM 立即回到該 Snapshot 的狀態。

**刪除 Snapshot**：

- 在 Snapshot Manager 中選擇目標 → `Delete`。
- 刪除 Snapshot 不會影響 VM 目前的狀態，只是合併差分磁碟檔案、釋放空間。
- 刪除過程可能需要數分鐘（視差分磁碟大小而定），期間不建議操作 VM。

### 6.4　Snapshot 使用注意事項

- **命名規範**：建議使用「狀態描述 + 日期」格式，例如 `Clean Install 2026-03-12`，方便日後辨識。
- **數量控制**：Snapshot 越多，差分磁碟檔案越多，VM 效能會逐漸下降。課堂練習建議不超過 3–5 個。
- **磁碟空間**：每個 Snapshot 會產生差分磁碟檔案（`.vmdk` delta），隨著 VM 內的變更量增加而增大。注意 host 磁碟空間。
- **開機狀態 vs 關機狀態**：開機狀態下建立的 Snapshot 會同時保存記憶體內容（`.vmem` 檔案），回復後可直接恢復到執行中狀態；關機狀態下建立的 Snapshot 只保存磁碟，回復後需重新開機。

### 6.5　最小驗收證據

1. **Snapshot 建立**：Snapshot Manager 截圖（含名稱與時間）。
2. **狀態變更**：變更前後對比截圖（例如建立一個文字檔 → 確認檔案存在）。
3. **回復驗證**：Revert 後截圖，證明與建立時一致（例如該文字檔已消失）。
4. **文字說明**：簡述做了什麼變更、回復後觀察到什麼。

---

## 7　故障排除

### 7.1　常見問題速查

| 症狀                             | 最可能原因                    | 第一步                                       | 進階排查                                                                      |
| -------------------------------- | ----------------------------- | -------------------------------------------- | ----------------------------------------------------------------------------- |
| 安裝檔無法執行                   | 非管理員權限                  | 確認本機管理員身份                           | 右鍵 → 以系統管理員身分執行；檢查防毒軟體是否攔截                            |
| 安裝完成但功能不穩               | 未重開機                      | 重開機再驗證服務                             | 重開機後用 `services.msc` 確認所有 VMware 服務為「執行中」                  |
| 無法執行 64-bit guest            | VT-x / AMD-V 未啟用           | 檢查 BIOS / UEFI 虛擬化設定                  | 工作管理員 > 效能 > CPU 確認「虛擬化」欄位；同時確認 Hyper-V 是否衝突         |
| 主視窗可開但部分功能異常         | Workstation Server 服務未啟動 | 檢查服務狀態                                 | `services.msc` 中手動啟動 `VMware Workstation Server` 服務                |
| VM 開機極度緩慢                  | Hyper-V 共存導致效能降低      | 確認是否啟用 Hyper-V                         | 參見 2.2 節的 Hyper-V 共存說明                                                |
| Win11 guest 安裝卡在網路步驟     | NAT 情境的已知 issue          | 改用 Bridged 重試                            | 或在安裝畫面按 `Shift + F10` 開啟 CMD，輸入 `oobe\bypassnro` 跳過網路需求 |
| VM 內無法上網                    | 虛擬網路服務未啟動            | 確認 VMware NAT Service 和 DHCP Service 狀態 | `Virtual Network Editor` → `Restore Defaults` 重置虛擬網路               |
| 滑鼠無法在 host/guest 間自由切換 | 未安裝 VMware Tools           | 安裝 VMware Tools                            | `VM > Install VMware Tools`，在 guest 內執行安裝                            |

### 7.2　排錯決策樹

```
安裝檔打不開 / 安裝被拒
└─ 是本機管理員嗎？
   ├─ 否 → 切換管理員帳號重試
   └─ 是 → 防毒軟體攔截了嗎？
            ├─ 是 → 暫時停用即時防護後重試
            └─ 否 → 確認安裝檔完整（重新下載、比對檔案大小）

安裝完成但行為異常
└─ 已重開機嗎？
   ├─ 否 → 重開機再驗證
   └─ 是 → VMware 服務都在執行中嗎？（services.msc）
            ├─ 否 → 手動啟動服務，若啟動失敗則重新安裝
            └─ 是 → 檢查是否有舊版 VMware 殘留或 Hyper-V 衝突

無法建立 / 啟動 64-bit guest
└─ BIOS 的 VT-x 或 AMD-V 啟用了嗎？
   ├─ 否 → 啟用後重開機
   └─ 是 → Hyper-V 是否開啟？
            ├─ 是 → 關閉 Hyper-V 或確認 VMware 版本支援共存
            └─ 否 → 回到第 2 節檢核 Host OS 與相容性

Win11 guest 安裝卡在網路
└─ 把 VM 網路從 NAT 改為 Bridged
   ├─ 成功 → 安裝完成後可視需求改回 NAT
   └─ 仍失敗 → 在安裝畫面按 Shift+F10 執行 oobe\bypassnro 跳過網路需求

VM 內完全沒有網路
└─ host 本身可以上網嗎？
   ├─ 否 → 先修 host 網路
   └─ 是 → VMware NAT Service 在執行中嗎？
            ├─ 否 → 啟動服務或 Restore Defaults
            └─ 是 → guest 內的網路介面卡有拿到 IP 嗎？（ipconfig / ip addr）
                     ├─ 否 → guest 內重新取得 IP 或重啟網路介面
                     └─ 是 → 檢查 guest 防火牆設定
```

---

## 附錄 A　macOS 使用者替代路徑

Workstation Pro 只支援 Windows 與 Linux 主機。macOS 使用者應改走 VMware Fusion 生態（同樣為免費授權）。

- **Fusion Pro 官方文件**：[https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/fusion-pro/25H2/using-vmware-fusion.html](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/fusion-pro/25H2/using-vmware-fusion.html)
- **Fusion Pro Release Notes**：[https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/fusion-pro/25H2/release-notes/vmware-fusion-25h2-release-notes.html](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/fusion-pro/25H2/release-notes/vmware-fusion-25h2-release-notes.html)

---

## 附錄 B　OVF 匯入匯出

VMware Workstation 支援 OVF（Open Virtualization Format）格式的 VM 匯入與匯出，可用於在不同虛擬化平台之間搬移 VM（例如從 VirtualBox 匯入，或匯出給 VMware Fusion 使用）。

- **匯入 OVF VM**：[https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/workstation-pro/17-0/using-vmware-workstation-pro/creating-virtual-machines-in-workstation-user-guide/importing-and-exporting-virtual-machines-in-workstation-pro/import-an-open-virtualization-format-virtual-machine.html](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/workstation-pro/17-0/using-vmware-workstation-pro/creating-virtual-machines-in-workstation-user-guide/importing-and-exporting-virtual-machines-in-workstation-pro/import-an-open-virtualization-format-virtual-machine.html)
- **匯出 VM 到 OVF**：[https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/workstation-pro/25H2/using-vmware-workstation-pro/configuring-and-managing-virtual-machines/export-a-virtual-machine-to-ovf-format.html](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/workstation-pro/25H2/using-vmware-workstation-pro/configuring-and-managing-virtual-machines/export-a-virtual-machine-to-ovf-format.html)
