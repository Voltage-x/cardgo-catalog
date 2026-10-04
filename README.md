# cardgo-catalog

台灣信用卡回饋規則的開放資料。

## 目錄

| 檔案 | 內容 |
| --- | --- |
| `index.yaml` | 版本號與檔案清單。`version` 變大，App 才會當成新版本下載。 |
| `merchants.yaml` | 通路主檔。店名、別名集中在這裡，卡片規則只引用 `id`。 |
| `currencies.yaml` | 回饋幣別，以及 1 單位約當多少台幣。 |
| `cards/<id>.yaml` | 一張卡的方案、等級、通路群組與回饋規則。`source` 是當時參考的銀行公開頁。 |

## 這份資料是估計

費率依各卡 `source` 上的公開方案整理，用來跟其他卡比較，不是銀行官方檔。結算方式、通路認定、點數換算都可能和帳單不同；已知的簡化寫在該卡的 `caveat`。實際回饋以銀行公告與帳單為準。

## 改資料

1. 改規則、通路或新增卡片 yaml。
2. 新店名加在 `merchants.yaml`，卡片檔只寫已有的通路 `id`。
3. 把 `index.yaml` 的 `version` 加 1。

App 讀檔時會檢查欄位。打錯欄位名稱、引用不存在的通路、方案、等級、群組或幣別，整份目錄都不會套用，並列出是哪張卡、哪一條規則。

`index.yaml` 的 `format` 是格式版本。用到舊版 App 看不懂的寫法時要加 1，舊版 App 就會停在原本的目錄，不會讀錯。

## 欄位

### 通路（`merchants.yaml`）

| 欄位 | 說明 |
| --- | --- |
| `id`、`name` | 必填。`aliases` 是搜尋用的別名。 |
| `kind` | `brand` 店家（預設）、`mcc` 消費類型、`region` 地區、`rail` 支付方式。 |
| `region` | `domestic` 或 `overseas`，選這個通路時預設的國內外。 |
| `rail` | 支付方式的代號。`kind: rail` 必填；`card_present` 代表實體刷卡。 |
| `tags` | 分類標籤，例如 `[airline]`。卡片用 `tag:airline` 一次指到所有標了的通路。 |

### 卡片（`cards/<id>.yaml`）

| 欄位 | 說明 |
| --- | --- |
| `id`、`name`、`networks` | 必填。`networks` 是 `visa`、`mastercard`、`jcb` 等。 |
| `reward` | 預設回饋幣別，規則沒寫 `reward` 就用這個。 |
| `image` | 卡圖，相對於目錄根目錄，預設 `cards/<id>.png`。 |
| `effective` | `from`、`to`。規則沒寫自己的 `effective` 就沿用。 |
| `settlement` | `note` 顯示給使用者。`retroactive` 是改卡片設定時會跟著重算的範圍：`calendar_day` 回溯整天、`calendar_month` 回溯整月、`billing_cycle` 回溯本帳單週期（沒寫就是這個）。App 會提示這段期間內會少拿多少回饋。 |
| `tiers`、`flags` | 使用者自己選的等級（單選）與條件（可複選），都是 `{id, name}`。 |
| `schemes` | 方案。`pick_max` 可自選通路數、`pick_pool` 可選的群組、`lock_until` 日期或 `end_of_month`。 |
| `switches` | 切換次數限制：`max`、`period`、`actions`（`change_scheme`、`change_picks`）。 |
| `groups` | 通路清單，成員是通路 `id` 或 `tag:<標籤>`。 |
| `exclusions` | `any_merchant` 命中時，擋掉 `blocks` 種類（預設 `general`）的規則。 |

### 規則（`rules`）

| 欄位 | 說明 |
| --- | --- |
| `id` | 必填，同一張卡內不可重複。`label` 是顯示名稱。 |
| `kind` | `named` 指定通路（預設）或 `general` 一般消費。指定通路沒對上才輪到一般消費。 |
| `mode` | `instead` 互相取代、取最高（預設）；`extra` 疊加在上面。 |
| `when`、`unless` | 條件，全部成立才算。`unless` 成立時這條不適用。 |
| `rate`、`rate_by_tier` | 回饋率，`0.03` 是 3%。依等級不同就用 `rate_by_tier`。 |
| `cap` | 上限：`id`、`period`、`max_reward` 或 `max_reward_by_tier`。同 `id` 的規則共用額度。 |

`when` 和 `unless` 可用的條件：`schemes`、`tiers`、`networks`、`regions`、`rails`、`flags_all`、`flags_none`、`any_merchant`。`flags_all: [holiday]` 依 `dates/` 的行事曆判斷國定假日。

`any_merchant` 可以寫通路 `id`、`group:<群組>`、`tag:<標籤>`，或 `picked`（使用者在方案裡自選的通路）。

`period` 可用 `calendar_day`、`calendar_month`、`calendar_quarter`、`billing_cycle`（依使用者的結帳日）。

一筆消費超過 `instead` 規則的上限時，超過的部分會改用下一條對得上的 `instead` 規則計算，例如「達上限後改 1%」。

重複的內容可以用 YAML anchor：第一次寫 `effective: &autumn {from: "2026-10-01", to: "2026-11-30"}`，之後寫 `effective: *autumn`。
