# 小店 POS — 專案說明（給 Claude Code）

類似 iCHEF 的餐廳 POS，網頁版，給自己（或朋友）的一家店使用。介面和註解都用繁體中文。

## 已定案的需求
- 使用情境：內用、先吃後付，有桌位。
- 設備：1 台 iPad（從 Safari 加入主畫面，當成 PWA 使用）、1 台 Star mC-Print3 出單機（iCHEF 支援的型號為 MCP30／MCP31L／MCP31CI）。
- 出單方式：Star WebPRNT，透過店內網路直連。iPad 直接 POST 到 `http(s)://<印表機IP>/StarWebPRNT/SendMessage`，不經過伺服器。
- 資料：只存在 iPad 的 localStorage（key `pos.v1`），沒有後端；靠匯出／匯入 JSON 備份。
- 出單時機：送單時印廚房單（加點只印新加的品項，標【加點】；退菜印【退菜】單），結帳時印收據。
  - 廚房單：最上方用反白特大字印桌號。
  - 收據：只印店名。
- 付款：現金（含找零計算、可開錢櫃）、信用卡（只記錄付款方式，不串接刷卡機）。
- 不開統一發票。
- 菜單：分類、品項、口味／加料群組（必選或可不選、單選或複選、可設定加價）、備註、售完開關。
- 桌位圖：分區域。
  - 可以拖曳移動，也可以縮放（拖右下角的圓點或用滑桿），圓桌保持正圓。
  - 位置和大小以畫布百分比儲存，畫布固定 16:10。
- 報表：日營業額、現金和信用卡分開、折扣、品項銷量；可補印、作廢訂單、印日結單。
- UI 風格：液態玻璃（backdrop-filter 毛玻璃加高光）、帶彈性的滑動指示（`.lens` 加 `data-lens`）、View Transitions 換頁過場，有 CSS 動畫備援。

## 檔案
- `index.html`：全部程式都在這裡（原生 JS，沒有框架、沒有建置步驟）。
  - 主要區塊：資料與 seed／migrate → 共用 UI（modal、toast）→ 出單（WebPRNT XML、預覽）→ 各畫面的 render 函式 → `ACT` 事件表。
- `sw.js`：network-first 離線快取。**每次發布新版要把 `CACHE` 版號加 1**（目前是 `pos-v2`）。
- `manifest.webmanifest`、`icon-*.png`：PWA 用。
- `README.md`：給店家的安裝與操作說明（GitHub Pages、iPad 加入主畫面、出單機 HTTPS 憑證）。

## 慣例
- 狀態放在全域 `S`（會存檔）和 `V`（畫面狀態）。改資料後呼叫 `save()` 再 `render()`。
- 換頁要包在 `navigate(() => {...})` 裡，才會有過場動畫。
- 資料結構改版時，在 `migrate()` 補上舊資料的轉換，不能讓使用者的舊資料壞掉。
- 出單內容用 `J.txt/J.align/J.feed/J.cut/J.drawer` 組成 job，再交給 `printJob(job, label)`。沒設定印表機 IP 時會顯示預覽。
- 一行 48 個半形字（80mm 紙），中日韓全形字算 2 格（`cw()`）。

## 尚未驗證／待辦
- [ ] **WebPRNT 格式還沒在實機上測過**。目前的指令是照 Star 的規格自己寫的，沒有用官方的 StarWebPrintTrader.js。要驗證的項目：
  - `codepage="utf8"` 印中文是否正常；
  - `width`／`height` 放大和 `invert` 反白是否正確；
  - `<peripheral>` 能不能開錢櫃；
  - 回應裡的 `<success>` 能不能正確解析。
  - 不行的話，考慮改用官方 SDK 的 JS。
- [ ] iPad 信任出單機 HTTPS 憑證的實際操作流程，確認後補進 README。
- [ ] 在較舊的 iPad 上檢查毛玻璃的效能；太卡的話，把 `.item` 的 backdrop-filter 拿掉。
- [ ] 部署：GitHub Pages。
