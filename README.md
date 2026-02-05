# LPSM 2DID Scanning System (Frontend)

![Vue.js](https://img.shields.io/badge/vuejs-%2335495e.svg?style=for-the-badge&logo=vuedotjs&logoColor=%234FC08D)
![Flexium](https://img.shields.io/badge/Flexium-Manufacturing-orange?style=for-the-badge)

**LPSM 2DID 掃描系統** 是一個專為產線設計的可視化人機介面 (HMI)。本專案基於 **Vue 3** 開發，透過 WebSocket 與本地 Python Control Hub 及 PLC 進行實時通訊，並介接 MES 系統進行製品驗證與資料上傳。

## ✨ 主要功能 (Key Features)

* **實時硬體整合 (Real-time Hardware Integration)**
    * 透過 WebSocket 接收 **PLC** 訊號（上/下台面、進/出料狀態）。
    * 接收來自相機或掃描槍的 **2DID** 條碼數據。
* **嚴格的流程卡控 (Strict Process Control)**
    * **防混料**：自動比對工單與品目，防止錯誤製品流入。
    * **防重複**：透過前端 `productList` 狀態管理，防止同一製品重複計數或重工。
    * **防跳站**：驗證製品當前站點與製程順序。
    * **防干擾**：具備相機訊號防抖動 (Debounce) 機制。
* **動態可視化介面 (Dynamic Visualization)**
    * 使用 CSS 3D Transform (`perspective`) 模擬機台實際作業區與放料區的空間感。
    * 動態顯示左/右相機掃描狀態（OK/NG/Waiting）。
    * 針對長文字（如越南文）優化的 Flexbox 排版。
* **多語言支援 (i18n)**
    * 支援 **中文 (Traditional Chinese)** 與 **越南文 (Tiếng Việt)** 即時切換。
* **完善的異常處理**
    * MES API 斷線重試與錯誤提示。
    * WebSocket 心跳機制 (Heartbeat) 確保連線穩定。

## 🛠️ 技術棧 (Tech Stack)

* **Framework:** Vue.js 3 (Composition API, `<script setup>`)
* **Styling:** CSS3 (Variables, Flexbox, Grid, 3D Transforms)
* **Communication:**
    * WebSocket (與本地 Python Middleware 通訊)
    * Fetch API (與 MES/OIS Server 通訊)
* **State Management:** Vue Reactivity (`ref`, `reactive`, `computed`)

## 🚀 安裝與執行 (Installation)

### 前置需求
* Node.js (建議 v16+)
* 本地 Python Control Hub (需另外啟動，預設 Port: `8181`)

### 步驟

1.  **Clone 專案**
    ```bash
    git clone https://github.com/Peter-1119/2DID-Frontend.git
    cd 2DID-Frontend
    ```

2.  **安裝依賴**
    ```bash
    npm install
    ```

3.  **開發模式啟動**
    ```bash
    npm run dev
    ```

4.  **生產環境打包**
    ```bash
    npm run build
    ```

## ⚙️ 設定 (Configuration)

目前的 API 端點與 WebSocket 地址設定於 `script setup` 頂部，建議依據部署環境調整：

```javascript
// App.vue
const API_URL = "[http://10.1.5.122/gxfirstOIS/gxfirstOIS.asmx/GetOISData](http://10.1.5.122/gxfirstOIS/gxfirstOIS.asmx/GetOISData)";
const OIS_API_BASE = "[http://10.1.5.124:2151/api](http://10.1.5.124:2151/api)";
const wsUrl = 'ws://127.0.0.1:8181';
```