# Lab52 庫存系統 (lab52_inventory.html)

內部庫存管理工具，整合台灣倉、美國外倉、FBA 三個來源，自動計算可售月數與補貨建議。

---

## 使用方式

用瀏覽器開啟 `lab52_inventory.html`，上傳三份報告即可自動更新。

| 來源 | 檔案格式 | 取得方式 |
|------|----------|----------|
| 台灣倉 | `.xls` | ezlogistics → 貨品庫存查詢 → 轉 Excel |
| FBA | `.csv` | Seller Central → Reports → Fulfillment → Inventory → Daily Inventory History |
| 美國外倉 | `.xlsx` | AMZLG&S 檔案（系統自動讀 Inventory sheet） |

> 數據儲存在瀏覽器 localStorage，同一台電腦重開網頁資料會保留。

---

## 頁籤說明

| 頁籤 | 內容 |
|------|------|
| 庫存總覽 | 所有 SKU 整合數據，含 FBA 週數與總庫存月數 |
| 補貨建議 | 自動計算 FBA 補貨量與採購下單量 |
| 在途管理 | 台灣→美國在途手動填入；外倉→FBA 在途自動抓 FBA CSV |
| 美國外倉 | 外倉件數與最近效期 |
| FBA | 各 SKU 可售、在途、保留數量 |
| 台灣倉 | 批號、數量、效期、品質 |

---

## 補貨邏輯

### FBA 補貨（外倉 → FBA）
- **觸發條件**：FBA 可售 < 6 週
- **建議補量**：補到 3 個月庫存
- **公式**：`目標量(月銷×3) - FBA現有 - FBA在途`

### 採購補貨（下訂單給工廠）
- **觸發條件**：總庫存（台灣倉+外倉+FBA+在途）< 4 個月
- **建議補量**：補到 6 個月（含 2 個月生產週期）
- **公式**：`目標量(月銷×6) - 全通路總庫存`

---

## 效期警示

| 顏色 | 條件 |
|------|------|
| 橘色 △ | 距效期 < 180 天（6 個月） |
| 紅色 ⚠ | 距效期 < 90 天（3 個月） |

> 台灣倉和美國外倉顯示效期警示，FBA 不追蹤效期。

---

## EAN 對照表

### 單支噴霧
| EAN | 商品名稱 | 月銷量 |
|-----|----------|--------|
| 4714781020015 | Kids Oral Spray Strawberry | 2,400 |
| 4714781020022 | Kids Oral Spray Watermelon | 1,100 |
| 4714781022224 | Kids Oral Spray Grape | 1,200 |
| 4714781022231 | Kids Oral Spray Vanilla | 950 |
| 4714781022248 | Kids Oral Spray Flavor Free | 700 |
| 4714781020756 | Kids Oral Spray Peach | 200 |

### 雙入噴霧
| EAN | 商品名稱 | 月銷量 |
|-----|----------|--------|
| 4714781023580 | Kids Oral Spray Strawberry+Grape | 450 |
| 4714781023603 | Kids Oral Spray Strawberry+Vanilla | 250 |
| 4714781023597 | Kids Oral Spray Strawberry+Peach | 250 |
| 4714781023412 | Kids Oral Spray Strawberry+Watermelon | 200 |

### 三入噴霧
| EAN | 商品名稱 | 月銷量 |
|-----|----------|--------|
| 4714781023122 | Kids Oral Spray Strawberry Trio | 150 |
| 4714781023115 | Kids Oral Spray Grape Trio | 130 |

### 成人噴霧
| EAN | 商品名稱 | 月銷量 |
|-----|----------|--------|
| 4714781020084 | Probiotic Oral Spray Apple Mint | 700 |

### 牙膏
| EAN | 商品名稱 | 月銷量 |
|-----|----------|--------|
| 4714781020060 | Kids Toothpaste Strawberry | 150 |
| 4714781020077 | Kids Toothpaste Watermelon | 150 |

---

## 特殊 EAN 對照

| 原始 EAN | 對應到 | 說明 |
|----------|--------|------|
| 4714781022217 | 4714781020015 | 舊版草莓，貼標後併入正版 |
| 4714781022255 | 4714781020756 | 舊版 Peach，已停用 |

---

## FBA SKU 對照表

| FBA SKU | EAN |
|---------|-----|
| AD-0001 | 4714781020084 |
| Grape_Spray_Kids | 4714781022224 |
| Milk_Kids_Spray | 4714781022231 |
| FlavorFree_Spray_Kids | 4714781022248 |
| Peach_Spray_Kids | 4714781020756 |
| Stawberry_Spray_Kids | 4714781020015 |
| StrawberryNHAP | 4714781020060 |
| WatermelonNHAP | 4714781020077 |
| Watermelon-spray | 4714781020022 |
| D6-1MWN-Q0EN | 4714781023580 |
| M2-6V1S-09J4 | 4714781023603 |
| F8-1THT-CLFU | 4714781023115 |
| TU-V4OB-07QB | 4714781023597 |
| Strawberry+Watermelon | 4714781023412 |
| Peach-spray / PEACH-SPRAY | 4714781020756 |
| CG-3XYS-TGEJ | 4714781023122 |
| StrawberrySprayBox / XI-MN9Y-L5CY | 4714781020015 |

---

## 如何修改這個系統

### 更新月銷量
在 `lab52_inventory.html` 找到 `const CATALOG`，修改對應 EAN 的 `monthly` 數值。

### 新增 SKU
在 `CATALOG` 陣列新增一筆：
```javascript
{ean:'新EAN',name:'商品名稱',cat:'單支噴霧',monthly:月銷量},
```
同時在 `FBA2EAN` 新增 FBA SKU 對照。

### 調整補貨門檻
- FBA 補貨觸發：找 `fbaWeeks(i)<6` 改數字（單位：週）
- 採購觸發：找 `totalMonths(i)<4` 改數字（單位：月）
- FBA 目標量：找 `i.monthly*3` 改倍數
- 採購目標量：找 `i.monthly*6` 改倍數

### 需要 AI 協助修改
把 `lab52_inventory.html` 上傳給 Claude，說明要改什麼即可。
不需要舊的對話記錄，檔案本身包含所有邏輯。

---

## 品牌色彩
| 色名 | Hex |
|------|-----|
| Light Powder Petal（背景）| #f5efec |
| Dark Slate Grey（主色）| #234f4b |
| Tropical Teal（強調）| #45bac4 |
| Dust Grey（分隔）| #e3d7d1 |
| Khaki Beige | #b19d8e |

---

*最後更新：2026/05*
