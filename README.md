# FY26 PL 分析儀表板 — Benesse Taiwan

單檔 HTML 分析工具，供財務資深分析師每月更新 PL 數據、查看各事業損益與推估、匯出後推送至 GitHub 永久保存。

---

## 技術架構

| 項目 | 說明 |
|------|------|
| 執行環境 | 純前端單檔 HTML（無伺服器、無框架） |
| 圖表 | Chart.js 4.4.3 + chartjs-plugin-datalabels 2.2.0 |
| Excel 解析 | SheetJS (xlsx 0.18.5) |
| 資料持久化 | `DATA` 物件 → `exportHTML()` 嵌入 HTML → GitHub commit |
| 語言切換 | 中文（zh-TW）/ 日文（ja），i18n 字典內建 |
| 貨幣切換 | NTD / JPY，匯率從 Excel 匯入 |

---

## 頁面結構

| 頁面 | ID | 說明 |
|------|----|------|
| 經營摘要 | `page-summary` | 5 KPI tiles → Bridge chart → Donut+Margin → P&L 表 → 洞察文字 |
| 總覽 | `page-overview` | 全事業收支總表、流量分析 |
| 事業別 | `page-business` | 各事業 PL 詳細、瀑布圖、KPI、預測面板 |
| 共通費用 | `page-common` | 共通費用明細與推估 |
| KPI | `page-kpi` | KPI 月次追蹤 |
| 月次比較 | `page-monthly` | 單月 / 多月橫向比較 |
| 廠商分析 | `page-vendor` | 廠商別收入貢獻 |

---

## 事業清單

```
學習商品 / 舞台劇 / 授權金 / 周邊商品 / 巧連智數位經典 / Mirafeel / 其他 / 日本跨境進口A
```
加上「共通」（共通費用）= ALL_BIZ（共 9 個）。

---

## 資料流程

### 每月更新步驟

1. **匯入 Excel**（頁面右上角 ⬆ 匯入按鈕）
   - `Benesse_Taipei_Banch_PL.xlsx`：PL 預算 + 實績 + FY25 實績
   - `FY26_KPI.xlsx`：KPI 數據
   - 匯率工作表（`財務年月` / `匯率` 欄）

2. **手動輸入**
   - **Adj.EBITDA**：點擊「經營摘要」頁第 5 個 KPI tile（藍色底線），輸入 JPY 百万數字
   - 公式：`Adj.EBITDA = 營業利潤 + 特損 + 折舊 − 資本支出`（來源另存 Excel，手動貼入）
   - **洞察文字**：點擊「洞察・課題」區域直接編輯

3. **匯出 HTML**（右上角 ⬇ 匯出 HTML 按鈕）
   - 所有 DATA（含 Adj.EBITDA、匯率、洞察文字）嵌入 HTML
   - 覆蓋 `fy26_pl_dashboard.html`，commit + push 即永久儲存

### DATA 持久化欄位

| 欄位 | 型態 | 說明 |
|------|------|------|
| `DATA.adj_ebitda_jpy` | `number \| null` | Adj.EBITDA（JPY 百万），手動輸入 |
| `DATA.exchange_rate` | `{YYYYMM: number}` | NTD→JPY 匯率（從 Excel 匯入） |
| `DATA.issue_notes` | `{biz: string}` | 各事業課題備注 |
| `DATA.insight_note` | `string` | 經營摘要洞察文字 |
| `DATA.budget_init` | nested object | FY26 預算（從 Excel 匯入） |
| `DATA.actual` | nested object | FY26 實績（從 Excel 匯入） |
| `DATA.fy25_actual` | nested object | FY25 實績（從 Excel 匯入） |
| `DATA.kpi` | object | KPI 數據（從 Excel 匯入） |

---

## 推估邏輯（`calcForecast`）

- 有實績月份：使用實績數字
- 無實績月份：使用預算數字
- 比較基準：預算 vs 調整後預算（`STATE.compareBase`）
- 全事業加總：`calcForecastAll(excludeList=[])`

---

## Traffic Light 規則

| 燈號 | 條件 | 指標 |
|------|------|------|
| 🟢 | 達成率 ≥ 95% | 當月收入 or 全年推估收入 |
| 🟡 | 85% ≤ 達成率 < 95% | 同上 |
| 🔴 | 達成率 < 85% | 同上 |

每個事業顯示兩個燈號（左：當月、右：全年推估）。

---

## 「經營摘要」頁架構（tabA 風格）

參考 FY25 年報 tabA 設計，由上而下共 5 層：

1. **5 KPI Tiles**：總收入 / 事業利潤（扣除共通前）/ 共通費用 / 營業利潤 / Adj.EBITDA（可點擊輸入）
2. **Bridge Chart**：水平瀑布圖，各事業利潤貢獻 → 事業計 → 共通費逐項 → 營業利潤
3. **Donut + Margin**：收入構成圓餅圖 + 事業別利潤率橫向長條圖
4. **P&L 摘要表**：事業｜收入｜原價｜原價率｜營業費用｜人件費｜一般管費｜營業利潤｜利潤率（含 TL 燈號）
5. **洞察・課題**：可直接點擊編輯的富文字區塊

---

## Git 工作流程

```bash
# 開發分支
git checkout claude/tender-hopper-zPp0Z

# 匯出 HTML 後覆蓋檔案，再 commit
git add fy26_pl_dashboard.html
git commit -m "FY26 YYYYMM 更新：匯入實績 / 調整預測"
git push -u origin claude/tender-hopper-zPp0Z
```

---

## 本地開啟方式

直接在瀏覽器開啟 `fy26_pl_dashboard.html`（File → Open），無需本地伺服器。

---

## 檔案清單

| 檔案 | 說明 |
|------|------|
| `fy26_pl_dashboard.html` | 主儀表板（唯一交付物） |
| `pl_data.json` | 開發用資料草稿（非正式，已由 DATA 內嵌取代） |
| `README.md` | 本文件 |
