# cardgo-catalog

台灣信用卡回饋規則的開放資料。

[Cardgo](https://github.com/Voltage-x/cardgo) 用這份目錄，在到店或搜尋通路時，依手上卡片的方案、等級與回饋上限，排出估計回饋。App 本體目前還沒有開源。規則、通路和點數匯率獨立放在這裡，可以單獨更新，也不綁在 App 程式裡。

Cardgo 把這個 repo 掛在 `catalog/`（git submodule），並把同一份檔案打包進 App。啟動時若設定了目錄網址，會在背景讀取 `index.yaml` 的 `version`；比內建的新，就下載其餘檔案。沒有網路時沿用打包進去的那份。

遠端目錄：

```
https://raw.githubusercontent.com/Voltage-x/cardgo-catalog/main/catalog
```

## 目錄

| 檔案 | 內容 |
| --- | --- |
| `catalog/index.yaml` | 版本號與檔案清單。`version` 變大，App 才會當成新版本下載。 |
| `catalog/merchants.yaml` | 通路主檔。店名、別名集中在這裡，卡片規則只引用 `id`。 |
| `catalog/currencies.yaml` | 回饋幣別，以及 1 單位約當多少台幣。 |
| `catalog/cards/<id>.yaml` | 一張卡的方案、等級、通路群組與回饋規則。`source` 是當時參考的銀行公開頁。 |

## 這份資料是估計

費率依各卡 `source` 上的公開方案整理，用來跟其他卡比較，不是銀行官方檔。結算方式、通路認定、點數換算都可能和帳單不同；已知的簡化寫在該卡的 `caveat`。實際回饋以銀行公告與帳單為準。

## 改資料

1. 改規則、通路或新增卡片 yaml。
2. 新店名加在 `merchants.yaml`，卡片檔只寫已有的通路 `id`。
3. 把 `catalog/index.yaml` 的 `version` 加 1。
