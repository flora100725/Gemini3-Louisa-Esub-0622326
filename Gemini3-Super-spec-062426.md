# 醫療器材 FDA 510(k) 智能審查報告門戶與 Agentic 系統設計技術規格書
## (Technical Specification for FDA 510(k) Regulatory Compliance Agentic Portal)

---

## 1. 執行摘要 (Executive Summary)

本技術規格書旨在定義一套前瞻性、AI 驅動的醫療器材 **FDA 510(k) 智能審查報告門戶系統**。此系統專為高階醫學診斷設備（如超音波診斷系統 **Samsung Medison HERA Z20**，對標已核准案號 **K241971**）之法規專家、研發團隊與審查委員設計。

本系統採用「**Agentic（代理人）架構**」作為核心設計理念，不再是傳統的靜態表單，而是由主動式 AI 代理引導用戶完成 30 道涵蓋電性安全、電磁相容、生物相容性、軟體確效、網路安全與 AI CADe/CADx 演算法的關鍵法規審查問題。隨後，系統將基於使用者的結構化與非結構化回答，自動合成出符合 FDA 規範、長達 3,000 至 4,000 字的 **Traditional Chinese（繁體中文）510(k) 技術審查報告**。

本系統更融合了「**雙欄同步對照編輯器**」、「**多格式輸出引擎 (Google Docs, PDF, HTML, Markdown)**」、「**問題庫動態導入/導出模組**」，以及共計 **8 項極具震撼力的「Wow AI 魔法功能」**（包括 5 項基礎 AI Magic 與 3 項追加先進 AI 功能）。

---

## 2. 系統架構與設計理念 (System Architecture & Design Philosophy)

### 2.1 系統拓撲與技術棧 (System Topology & Tech Stack)
系統採用基於 **React 19 + TypeScript + Vite** 的全端架構（透過內嵌的 **Express.js** 作為 API 閘道），確保前端 UI 的高響應性與後端資料處理及 API 金鑰的安全性。

```
+---------------------------------------------------------------------------------+
|                                 Browser (Client)                                |
+---------------------------------------------------------------------------------+
|                                                                                 |
|  +--------------------+     +---------------------------+     +--------------+  |
|  | Agentic Side Panel |     | Sync Markdown Editor Pane |     | Power Bar    |  |
|  | (30-Q Interactive) |     |  (Markdown VS Live HTML)  |     | (8 AI Magics)|  |
|  +---------+----------+     +-------------+-------------+     +------+-------+  |
|            |                              |                          |          |
+------------|------------------------------|--------------------------|----------+
             | REST API / Event Stream      | Synchronous Scroll / DOM | REST API
             v                              v                          v
+---------------------------------------------------------------------------------+
|                                  Express Server                                 |
+---------------------------------------------------------------------------------+
|                                                                                 |
|  +---------------------------------------------------------------------------+  |
|  | API Proxy & Security Gateway                                              |  |
|  | - Lazy-initialized Google Gen AI Client (process.env.GEMINI_API_KEY)      |  |
|  | - Workspace Integration Node (Google Docs API & OAuth Gateway)             |  |
|  | - PDF/HTML Template Compiler & MD Parser                                  |  |
|  +---------------------------------------------------------------------------+  |
|                                                                                 |
+----------------------------------------+----------------------------------------+
                                         | Secure SSL Connection
                                         v
                      +--------------------------------------+
                      |      External API Providers          |
                      | - Google Gemini 2.5 Flash / Pro      |
                      | - Google Workspace (OAuth / Drive)   |
                      +--------------------------------------+
```

### 2.2 資訊安全與 API 金鑰防護 (Security & API Key Isolation)
依據安全性規範，**所有敏感金鑰（如 `GEMINI_API_KEY`、`GOOGLE_CLIENT_SECRET`）一律不曝露給前端瀏覽器**。
*   **伺服器端代理 (Server-Side Proxy)**：前端所有 AI 生成請求、魔法功能呼叫與多格式導出，均發送至 Express 伺服器端的 `/api/ai/*` 與 `/api/export/*` 路由。
*   **延遲初始化 (Lazy Initialization)**：Google GenAI SDK 僅在端點首次被觸發時讀取環境變數並建立實例，防止環境變數未到位時造成容器啟動崩潰（Startup Crash）。
*   **OAuth 憑證安全**：與 Google 雲端硬碟及 Google Docs 整合時，採用標準授權碼流程（Authorization Code Flow），並透過 `httpOnly` 與 `Secure` 的 Cookie 來儲存短期 Access Token。

### 2.3 核心狀態引擎與本地持久化 (State Engine & Local Persistence)
*   **狀態結構設計 (`/src/types.ts`)**：
    *   `QuestionStore`: 儲存 30 道問題的結構、提示說明、當前答案與完成狀態。
    *   `ReportMetadata`: 儲存當前正在審查的產品（如 HERA Z20）基礎規格與適應症。
    *   `EditorState`: 控制雙欄編輯器中的 Markdown 原生文字、HTML 渲染樹與游標位置。
*   **本地備援 (Local Backup)**：採用 `localStorage` 作為用戶輸入資料的即時快取防失機制。使用者在回答 30 道問題或修改報告時，系統每 3 秒自動將狀態序列化並寫入 `localStorage`，防範瀏覽器無預警崩潰或連線中斷。

---

## 3. 30 道核心法規審查問題模組 (30-Question Regulatory Framework)

系統設計一個 **Interactive Question Module**，透過結構化問題收集擬申報器材（本案為 HERA Z20）的關鍵技術檔案參數。這 30 道問題根據 FDA 510(k) 審查實務與指引文件分類：

### 3.1 問題庫分類與設計清單

| 編號 | 審查範疇 (Domain) | 核心問題內容 (Traditional Chinese) | 法規指引依據 | 預期輸入類型 |
|:---:|:---|:---|:---|:---|
| 1 | **產品基本規格** | 請問本案超音波系統之型號、主要適應症（預期用途）及目標患者族群為何？是否與 K241971 完全一致？ | K241971 / FDA Intended Use | 自由文字 (Text) |
| 2 | **產品基本規格** | 請列出所有擬申請上市之超音波探頭型號（如 CMV1-10, CMV1-10Z）及其基本物理架構。 | FDA Ultrasound Guidance | 列表 (List/Text) |
| 3 | **產品基本規格** | 探頭之掃描類型（如凸陣、線陣、相控陣、微凸陣、經食道等）與掃描角度或掃描長度規格為何？ | FDA Ultrasound Guidance | 結構化數值 |
| 4 | **產品基本規格** | 請說明本案產品所支援之所有操作模式（如 2D, M, Color, PW, CW, 3D/4D, Elastoscan+ 等）。 | 中文說明書規格 | 多選核取方塊 |
| 5 | **電性安全 (ES)** | 本案是否檢附最新 IEC 60601-1:2025 之測試報告？出具之第三方實驗室資歷與證書是否完整？ | IEC 60601-1:2025 | 文本與檔案確認 |
| 6 | **電磁相容 (EMC)** | 產品是否符合 IEC 60601-1-2:2024 (或2014) 電磁相容測試標準？是否有漏電流或耐受性異常記錄？ | IEC 60601-1-2:2024 | 文本/合格確認 |
| 7 | **超音波安規 (Particular)** | 針對 IEC 60601-2-37（超音波特定安全要求），本案主機及所有探頭是否已取得完整測試評估報告？ | IEC 60601-2-37 | 文本/合格確認 |
| 8 | **聲學輸出安全** | 聲學輸出測量是否依據 IEC 62359 或 NEMA UD 2-2004 標準進行？是否檢附各探頭最大聲學輸出數據？ | IEC 62359 / NEMA UD 2 | 數值與文本 |
| 9 | **生物相容性** | 所有探頭之接觸性質（接觸皮膚、黏膜或手術中接觸）及接觸時間（短暫、中期、長期）如何分類？ | ISO 10993-1 分類 | 下拉選單 |
| 10 | **生物相容性** | 是否針對所有探頭提供細胞毒性、致敏性及刺激性（ISO 10993-5, -10）測試報告？若無，其豁免理由為何？ | ISO 10993-1 要求 | 自由文字 (評估) |
| 11 | **生物相容性** | 針對經直腸、經陰道及手術中等高風險探頭，其滅菌或高層次消毒（HLD）之驗證參數與相容性是否完整？ | FDA Reprocessing Guidance | 文本與滅菌參數 |
| 12 | **軟體確效 (SW)** | 本案軟體之「關注層級」（Level of Concern, LoC / 軟體安全級別）分類為何（Major, Moderate, Minor）？ | 醫療器材軟體確效指引 | 下拉選單 (A/B/C級) |
| 13 | **軟體確效 (SW)** | 是否檢附符合 IEC 62304 標準之軟體生命週期確效報告（包含 SRS、SDS、測試計劃及未解決臭蟲分析）？ | IEC 62304 / 軟體確效指引 | 文本/合格確認 |
| 14 | **網路安全 (Cyber)** | 本案是否提供網路安全自我評估報告？是否針對器材之資產、威脅及漏洞進行全面性威脅建模（Threat Modeling）？ | 醫療器材網路安全指引 | 自由文字 / JSON |
| 15 | **網路安全 (Cyber)** | 系統是否具備用戶身分驗證、加密傳輸（TLS 1.3）、系統稽核日誌及惡意軟體防護機制？ | 醫療器材網路安全指引 | 核取清單與描述 |
| 16 | **網路安全 (Cyber)** | 軟體資產清單（SBOM）是否包含所有第三方與開源組件？是否具備已知漏洞（CVE）之修補計劃？ | SBOM 規範 | 自由文字 / 列表 |
| 17 | **AI / CAD 功能** | 本案 AI 功能（如 ViewAssist, Live ViewAssist, HeartAssist, EzVolume）之預期用途與臨床定位為何？ | CADe/CADx 技術指引 | 自由文字 (預期用途) |
| 18 | **AI 演算法架構** | 該 AI 演算法之模型架構（如 CNN, RNN, Transformer）、開發框架及關鍵超參數設定為何？ | CADe/CADx 技術指引 | 技術參數描述 |
| 19 | **AI 訓練數據來源** | 訓練與驗證數據集之規模為何？數據來自哪些醫療機構？是否具備設備廠牌、影像品質之多樣性？ | 數據集規模與來源 | 數量統計與列表 |
| 20 | **AI 數據人口統計** | 數據集中患者的人種、年齡、性別、地理分布之比例為何？是否能代表擬申報之目標受試族群？ | 人口統計學特徵 | 數據比例與描述 |
| 21 | **AI 數據獨立性** | 如何確保測試數據（Test Data）與訓練數據（Training Data）的絕對獨立性？是否採用三折/五折交叉驗證？ | 數據獨立性與泛化能力 | 隔離策略描述 |
| 22 | **AI 數據標註程序** | 影像標註人員之專業資歷與臨床背景為何？如何解決多位標註者之間的一致性差異（Inter-rater Reliability）？ | 數據清洗與標註流程 | 標註標準與黃金標準 |
| 23 | **AI 性能評估指標** | AI 功能在獨立測試集上之靈敏度、特異度、AUC、ROC 曲線及混淆矩陣之具體數值為何？ | Non-Clinical Performance | 統計數值 (CI 95%) |
| 24 | **AI 統計信心度** | 系統輸出之統計信心水準（Confidence Intervals）或預測不確定性度量（Uncertainty Metrics）如何呈現給用戶？ | 統計信心與不確定性 | 輸出邏輯與置信度 |
| 25 | **AI 更新與維護** | 產品上市後，針對模型漂移（Model Drift）的監控機制及持續學習、版本迭代之驗證程序為何？ | 模型更新與維護機制 | 監控與再確效流程 |
| 26 | **最終品質與性能** | 各探頭之軸向解析度（Axial Resolution）與側向解析度（Lateral Resolution）之具體量測數值是否符合規格？ | Performance Test | 探頭規格數值 |
| 27 | **量測功能準確度** | 系統內建之各項量測類型（如距離、體積、心跳、流速等）其最大量測誤差與準確度是否符合臨床規範？ | Performance Test | 測量精準度列表 |
| 28 | **定量軟體確效** | 對於定量或半定量量測之特殊軟體功能，其演算法有效性與臨床測試對比之評估結果為何？ | 臨床前測試基準 | 測試結論描述 |
| 29 | **熱/機械指數** | 熱指數 (TI: TIS, TIB, TIC) 與機械指數 (MI) 之最大值是否符合 IEC 60601-2-37 規範？其限制機制為何？ | IEC 60601-2-37 | 數值與安全限值 |
| 30 | **臨床證據與補件** | 針對原廠技術資料之不完整處（例如生物相容性補件、探頭性能補件、AI 報告補件），本案之補件應答策略為何？ | 審查結果應答 | 綜合對策與策略 |

---

### 3.2 問題庫 Markdown 導入/導出與修改模組
為賦能用戶靈活調整審查框架，系統設計 **Markdown 格式問題庫解析引擎**。
*   **上傳模組 (Import)**：用戶可拖入或選擇一個標準的 `.md` 問題庫檔案。系統使用內置的正則解析器（Regex Parser），自動提取以 `### [Q_ID] Question` 開頭的區塊、問題描述與預期輸入類型。
*   **在線修改 (Modify)**：提供互動式網頁介面。用戶可以自由新增問題、調整既有問題的法規指引依據，或重新排序 30 道問題。
*   **下載模組 (Export)**：一鍵將當前正在作答或已客製化的問題庫導出為標準 Markdown。

**導入/導出問題庫樣式範例 (Questions Schema in Markdown)**：
```markdown
# FDA 510(k) 審查問題庫配置

### [Q_01] 產品基本規格與預期用途
* **法規指引依據**: K241971 / FDA Intended Use Guidance
* **預期輸入類型**: Text
* **詳細說明**: 請詳細描述主機與探頭之預期用途，並對標實體等效器材（Predicate Device）。

### [Q_09] 探頭生物相容性分類
* **法規指引依據**: ISO 10993-1:2018
* **預期輸入類型**: Dropdown (Skin, Mucosa, Surgical-Intraoperative)
* **詳細說明**: 定義所有探頭之接觸介面與接觸時間，以便推導測試矩陣。
```

---

## 4. 510(k) 審查報告生成引擎 (Review Report Generation Engine)

### 4.1 智能合成與動態插值
當使用者完成 30 道問題後（或在回答過程中點擊「即時生成」），系統啟動伺服器端的 **Agentic Report Synthesis Engine**。此引擎將 30 道問題的回答，與系統預置的 FDA 510(k) 審查標準範本（對標 HERA Z20、K241971、Nemko 測試報告等實例）進行融合。
*   **脈絡感知提示詞 (Context-Aware Prompting)**：利用 Gemini API 的 `systemInstruction` 灌輸 FDA 專業審查官之角色設定，依據用戶提供的零散數據、數值，擴展為格式精整、術語嚴謹的法規論述。
*   **補件邏輯分析**：對於用戶回答「無測試數據」或「不適用」的章節（如探頭解析度缺失、生物相容性未檢附），AI 引擎會自動生成「不符合 (NON-COMPLIANT)」之標記，並在審查結論中自動彙整補件應答建議（Deficiency Responses）。

### 4.2 珊瑚色新增內容標記之技術實現 (Coral-Highlight Rendering)
為使法規專家與審查委員一目了然，本案特別要求：**由使用者回答所推導或新增之內容，必須在最終的 Markdown 報告中以「珊瑚色 (Coral Color, #F87171)」呈現**。
*   **技術方案一：HTML 內嵌樣式標記**
    AI 在生成 Markdown 時，會將基於用戶回答動態演繹出的段落、數值或補件策略，包裹在 HTML 標籤中：
    `這款探頭在 <span style="color: #F87171; font-weight: 500;">軸向解析度 0.5mm 與側向解析度 0.8mm</span> 表現上...`
*   **技術方案二：自定義 Markdown 語法與前端解析**
    AI 使用雙底線加特定類別標記，例如 `==新增內容==` 或 `[coral](新增內容)`。前端 Markdown 渲染器（`react-markdown`）在解析時，註冊自定義渲染元件（Custom Component），將特定語法轉換成帶有 Tailwind 類別的 span：
    `class="text-coral-500 font-semibold highlight-coral"`（對應 `#F87171` 的微亮背景與文字偏色）。

---

## 5. 基礎 5 項 AI Magic 核心魔法功能 (5 Base AI Magic Features)

為了讓審查與補件工作達到「Wow」級別的極致流暢，系統在 Power Bar 中內建 5 項 AI 核心魔法功能。每項功能均配備專屬的提示詞管道與伺服器端智能代理。

```
+---------------------------------------------------------------------------------+
|                                 POWER BAR (16px Rail)                           |
+---------------------------------------------------------------------------------+
|   [⚖️] 1. Gap Analysis & Auto-Remediation (差異對比與自動修復)                    |
|   [🛡️] 2. Standards Alignment & Compliance Check (法規測試對齊)                 |
|   [🔥] 3. Acoustic Safety & Index Plausibility (聲學指標查驗)                   |
|   [🧬] 4. Biocompatibility Master File Check (生物相容性比對)                    |
|   [🔒] 5. Cybersecurity Threat Modeling (網路安全威脅建模)                     |
+---------------------------------------------------------------------------------+
```

### 5.1 差異對比與自動修復 (Gap Analysis & Auto-Remediation)
*   **功能描述**：一鍵掃描當前正在編輯之 510(k) 報告，比對美國 FDA 最新超音波系統審查指引，自動抓出未滿足的法規要素（Gaps），並直接在編輯器中生成「修復建議草稿」。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    你是一位極度嚴苛的 FDA 510(k) 首席醫療器材審查官（Lead Reviewer）。
    請分析以下 Markdown 審查報告。找出以下項目是否有遺漏：
    1. 探頭型號 CMV1-10 與 CMV1-10Z 缺失之軸向與側向解析度測試。
    2. 是否缺少 IEC 60601-2-37 聲場強度測試報告？
    請定位並列出所有 'Gap'，並針對每個 Gap 提供一段建議填導之技術段落，將該段落包裹在 `<span style="color: #F87171;">` 中。
    ```
*   **互動反饋**：點擊後，系統會在雙欄編輯器左側以紅色警示點（Glow Dots）標出 Gap，並在右側預覽區高亮顯示珊瑚色修復內容。

### 5.2 臨床前測試準則對齊與法規符合性檢查 (Standards Alignment & Compliance Verification)
*   **功能描述**：自動掃描並核對報告中引用的標準是否為最新版本（例如將舊版 IEC 60601-1 自動升級或對比為最新之 IEC 60601-1:2025 與 IEC 60601-1-2:2024），並指出報告內容是否滿足標準的特定條款（Clause）。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    請解析報告中引用的標準（IEC 60601-1, IEC 60601-1-2, ISO 10993-1）。
    比對以下最新標準版本與要求：
    - IEC 60601-1:2025 (取代舊版)
    - IEC 60601-1-2:2024 (取代 2014)
    標示出報告中引用的陳舊版本標準，並產出合規性轉換表格。將新增或修改之標準描述文字以珊瑚色 `#F87171` 高亮。
    ```
*   **互動反饋**：在側邊欄彈出「標準對比面板」，顯示當前版本 vs 最新版本的差異，提供「一鍵升級標準」功能。

### 5.3 聲學安全與熱/機械指數合理性查驗 (Acoustic Safety & Index Plausibility Check)
*   **功能描述**：審查所有探頭在各操作模式下之最大聲學輸出指標（熱指數 TI 限制為 $\le 6.0$，機械指數 MI 限制為 $\le 1.9$）。若用戶輸入的測量值超出安全阈值或不合常理，AI 會立刻發出物理學合理性警告，並給出降噪降功率的臨床前調整建議。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    請扮演超音波聲學物理專家。
    分析報告中 CMV1-10 等探頭之 MI (Mechanical Index) 與 TI (Thermal Index) 參數。
    - 檢查是否 MI > 1.9 或 TI > 6.0 之超標風險。
    - 檢查數值是否符合聲學測量標準 NEMA UD 2-2004。
    - 若數值不合理（例如 MI 數值與頻率、深度不成比例），指出漏洞並生成修正對策，並以珊瑚色包裹輸出。
    ```
*   **互動反饋**：在探頭規格表格上方，跳出黃色驚嘆號安全警示，並給出優化後的聲學係數估算。

### 5.4 生物相容性評估與主檔案比對 (Biocompatibility Assessment & Master File Check)
*   **功能描述**：針對 HERA Z20 擬配置之探頭接觸特性（如經陰道、經食道探頭接觸黏膜，屬於短期至中期接觸），檢查 ISO 10993-1 要求的細胞毒性、致敏性及刺激性試驗。AI 將檢索報告，確認是否需要檢附 Master File (MAF) 授權書，並生成豁免測試的法規合理性說明（Biocompatibility Rationale）。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    分析報告中所有探頭（特別是 CMV1-10 經直腸/陰道探頭）的接觸路徑。
    比對 ISO 10993-1 標準之測試矩陣。
    若探頭材料為已核准之 Predicate 器材同款，生成一段用於說服審查官的「材料實質等效豁免測試 Rationale」，並以珊瑚色標註，以便用戶直接插入補件回覆中。
    ```

### 5.5 網路安全防禦級別與威脅建模分析 (Cybersecurity Defense & Threat Modeling)
*   **功能描述**：針對 HERA Z20 超音波系統的連網與醫療影像傳輸功能（DICOM, Wi-Fi），自動執行威脅建模分析（STRIDE 模型），檢查是否具備安全認證、加密與漏洞修補計劃，並直接生成一段符合 FDA 2023 最新網路安全指引的補強文字。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    請扮演醫療器材網路安全漏洞專家。
    基於 HERA Z20 的網路功能（PACS 連線、DICOM、系統維護埠），使用 STRIDE 威脅建模框架分析其潛在威脅。
    生成一組 510(k) Cybersecurity 應答報告，包括 Spoofing, Tampering, Information Disclosure 等防禦措施，文字內的新配置參數與威脅修正對策以珊瑚色高亮顯示。
    ```

---

## 6. 追加 3 項先進「Wow」AI 魔法功能 (3 Additional Advanced Wow AI Features)

為了打破現有市面法規工具的侷限，我們額外設計了 3 項極具前瞻性與技術深度的 AI 魔法功能。

### 6.1 功能 6：臨床數據泛化能力與偏誤自動評估器 (Clinical Data Generalization & Bias Evaluator)
*   **核心痛點**：FDA 審查官最常質疑 AI 醫療器材（如 ViewAssist, S-Detect 等 CADe/CADx 功能）在多中心、多設備及不同患者群體上的「泛化能力 (Generalization)」，並常因「訓練數據偏誤 (Bias)」提出補件（Deficiency）。
*   **Wow 技術實現**：
    *   本功能會提取報告中「2-3 开发与验证数据集」及「2-4 独立性能评估」章節之數據。
    *   AI 代理會利用統計學公式（Confidence Intervals 信心區間）與人口統計多樣性矩陣，自動計算出一個 **「數據泛化指數 (Generalization Score, 0-100)」** 與 **「潛在偏誤熱圖數據 (Bias Heatmap Data)」**。
    *   AI 會自動評估：訓練數據中黑人/亞洲人比例是否不足？影像是否過度集中於單一醫療機構（如單一南韓醫院）？測試數據是否與訓練數據發生「資訊洩漏 (Data Leakage)」？
    *   評估完畢後，AI 會自動撰寫一段精密的「泛化能力合理性抗辯與未來上市後監控（Post-Market Surveillance）承諾書」，完全以 **珊瑚色** 渲染，助用戶化解審查官的疑慮。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    請扮演資深生物統計學家與 FDA 演算法審查委員。
    深度審查 HERA Z20 的 AI 模組（ViewAssist, S-Detect）之開發數據庫：
    - 計算人種、性別、年齡分布與目標人群（美國本土）的匹配偏誤。
    - 檢查測試集與訓練集的隔離度。
    - 產出數據偏誤分析報告，並以珊瑚色生成一段補償性臨床評估計畫與泛化能力抗辯 Rationale。
    ```

### 6.2 功能 7：FDA 510(k) 審查官詰問模擬器 (FDA Guidance & Examiner Interrogation Simulator)
*   **核心痛點**：遞交 510(k) 前，廠商最害怕收到 FDA 寄回的「AI/ML 補件清單 (Additional Information Request, AI Letter)」。
*   **Wow 技術實現**：
    *   點擊此功能後，系統界面會切換為一個模擬的 **「審查官聽證會議室」**。
    *   AI 代理將根據當前報告中寫得最模糊、數據最薄弱的 3 個章節，當場生成 **5 個辛辣且極具針對性的 FDA 審查官提問（Interrogations）**（例如：「您提到 CMV1-10 探頭在 2D 模式下的熱指數 TIC 達到 4.2，但並未提供對應顱骨骨骼受熱的安全限制說明，請解釋...」）。
    *   用戶可在模擬對話框中直接打字回答。AI 會即時分析用戶的應答，判斷「答覆滿意度得分」，並**一鍵將合格的答覆轉譯成標準法規論述，以珊瑚色高亮直接插入報告的對應章節中**。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    請扮演一位任職於 FDA 醫療器材評估與健康中心 (CDRH) 15 年的資深超音波安全審查組長。
    針對本份 HERA Z20 的報告，找出 3 個最容易被拒絕受理 (Refuse to Accept, RTA) 或要求補件 (AI) 的致命漏洞。
    生成 5 道極度尖銳、專業的問訊。當用戶回答後，將用戶口語回答擴增為極度嚴謹的珊瑚色 FDA 規格申辯文本。
    ```

### 6.3 功能 8：智能證據路徑圖與自動引用樹狀圖 (Smart Evidence Roadmap & Citation Tree Generator)
*   **核心痛點**：510(k) 技術檔案（Technical File）包含數百份子報告，審查官常因找不到「聲學輸出、安規、軟體確效與生物相容性」之間的交叉引用（Cross-Reference）而降低審查效率。
*   **Wow 技術實現**：
    *   此功能會解析整份 Markdown 報告，自動抽取出所有的「法規標準（如 IEC 60601-1-2）」、「實驗室報告編號（如 Nemko Report 2025）」、「探頭型號（CMV1-10）」與「軟體版本」。
    *   系統利用前端 **D3.js 或 Recharts** 引擎，動態繪製出一幅 **「互動式證據引用樹狀圖 (Smart Citation Tree Map)」**。
    *   樹狀圖會將標準與報告內文、附件報告建立邏輯連線。
    *   AI 代理更會自動在 Markdown 報告中植入一鍵連結與跳轉標記，並在有交叉引用風險的節點上（例如：硬體變更是否同步更新了軟體確效與網路安全評估？）自動生成提示說明，引用補強段落同樣以 **珊瑚色** 呈現。
*   **Agentic 提示詞設計 (Server-Side)**：
    ```
    請解析報告中的標準、報告編號、探頭、操作模式等實體。
    建立彼此之間的交叉引用矩陣 (Matrix)。
    若發現交叉引用矛盾（例如：生物相容性測試中的探頭材料名稱與主要規格章節不一致），生成紅色警告，並提供珊瑚色修正文字。
    ```

---

## 7. 多格式導出引擎與 Google Workspace 整合設計 (Multi-format Export & OAuth)

為滿足企業多元辦公場景，本系統建構了多格式文件轉換引擎：

```
+---------------------------------------------------------------------------------+
|                                 Export Engine Pipeline                          |
+---------------------------------------------------------------------------------+
|                                                                                 |
|                        +-----------------------+                                |
|                        |  Current Markdown State |                              |
|                        +-----------+-----------+                                |
|                                    |                                            |
|          +-------------------------+-------------------------+                  |
|          |                         |                         |                  |
|          v                         v                         v                  |
|   +--------------+          +--------------+          +--------------+          |
|   |   HTML-5     |          |  jsPDF-Auto  |          | Raw Markdown |          |
|   |  CSS-Inline  |          |  Table CSS   |          |  Plain Text  |          |
|   +------+-------+          +------+-------+          +------+-------+          |
|          |                         |                         |                  |
|          v                         v                         v                  |
|   Download .html            Download .pdf             Download .md              |
|                                                                                 |
|          +---------------------------------------------------+                  |
|          |  Google Docs API Pipeline (OAuth Secure Gateway)  |                  |
|          +-------------------------+-------------------------+                  |
|                                    |                                            |
|                                    v                                            |
|                    Real-time Upload to User Drive                               |
|                    and Auto-open Google Doc Link                                |
+---------------------------------------------------------------------------------+
```

### 7.1 HTML 導出 (帶有 CSS 樣式)
*   將 Markdown 編譯為帶有現代網頁視覺樣式的單一 HTML 檔案。
*   **珊瑚色保留**：導出的 HTML 將完整嵌入 Tailwind CSS 主題配色。所有經 AI 新增之內容將自帶 `class="highlight-coral"`（具有 `#F87171` 的亮眼背景與精緻字體邊框），確保離線閱讀依然保留審查軌跡。

### 7.2 PDF 導出 (前端 PDF 生成)
*   利用前端 `jsPDF` 與 `html2canvas` 引擎，或伺服器端 `Puppeteer` 將渲染好的 HTML 輸出成 A4 格式之 PDF。
*   **排版優化**：自動處理分頁斷點（Page-break Avoid），確保大表格（如 30 道問題的應答對照表、探頭規格表）不會被無情切斷。珊瑚色標記將在 PDF 中以高彩度粉紅灰色塊（15% 透明度 `#F87171` 背景）精確呈現，適合列印與呈報。

### 7.3 Markdown 導出 (標準格式)
*   一鍵下載原始 `.md` 文字檔。
*   **標籤保留**：其中經 AI 魔法修改的內容將保持標準 HTML 標籤包裹（如 `<span class="regulatory-added" style="color: #F87171;">...</span>`），使其在任何支援 Markdown 的編輯器中均能完美相容並正確變色。

### 7.4 Google Docs 導出與 OAuth 整合設計 (Workspace Integration)
此功能為企業用戶提供最直接的協作通道。
1.  **OAuth 授權工作流**：
    *   用戶點擊「匯出至 Google Docs」。
    *   系統調用 `set_up_oauth` 工具，請求 `https://www.googleapis.com/auth/documents` 與 `https://www.googleapis.com/auth/drive.file` 權限。
    *   前端安全重定向至 Google Consent Screen，用戶授權後返回授權碼。
2.  **文件格式轉譯與樣式注入**：
    *   伺服器端接收授權 Token，啟動 Google Docs API 管道。
    *   建立一個全新的 Google Doc。
    *   **Markdown 到 Google Docs 樣式映射**：將 `# Heading 1` 轉譯為 Google Docs 的 `HEADING_1` 樣式；將 `* bullet` 轉譯為無序清單結構。
    *   **珊瑚色樣式套用 (Text Styling)**：遍歷 Markdown 抽象語法樹（AST），當遇到標有 `#F87171` 或珊瑚色標記的文字區段時，API 會送出 `UpdateTextStyleRequest`，將該段文字的 `foregroundColor` 設為紅值 `248/255`、綠值 `113/255`、藍值 `113/255`（即 `#F87171`），並加上黃色微亮底色，使 Google Docs 檔案完美繼承審查印記，利於法規團隊進行後續的多人雲端協作編輯。

---

## 8. WOW 互動式網頁與 UI/UX 精緻化設計 (Interactive UI/UX Design)

本系統之網頁 UI 設計嚴格遵循「**Samsung Medison HERA Z20 | FDA 510(k) Compliance Portal**」的高端科技美學。

### 8.1 視覺識別與配色主題 (Visual Identity & Themes)
*   **主色調 (Dominant Colors)**：
    *   `Slate 900` (`#0F172A`)：用於深邃、專業的頂部導覽列與側邊欄背景，營造極度可靠的法規審查氣氛。
    *   `Slate 800` (`#1E293B`)：用於主容器與工作區背景。
    *   `Coral 500` (`#F87171`)：作為點睛之筆（Pop Color），用於 AI 智能代理狀態、進度條、高亮文字及核心呼叫按鈕（CTA）。
    *   `Canvas` (`#F8FAFC`)：工作區主底色，降低視覺疲勞。
*   **字體搭配 (Typography Pairs)**：
    *   標題與 display 文字：選用 **Inter / Space Grotesk**，呈現科技、專業感。
    *   正文與報告編輯區：選用具備極佳長文閱讀性的 **Merriweather (Serif, 襯線體)**，模擬真實 FDA 公文的閱讀體驗。
    *   系統參數、代碼與數值：選用 **JetBrains Mono**。

### 8.2 頁面三大區塊黃金比例排版
採用 `h-screen overflow-hidden` 佈局，完美將瀏覽器視窗劃分為三個互不干擾、各自滾動的現代「Bento 網格」區塊：

```
+---------------------------------------------------------------------------------+
|                                 1. HEADER                                       |
| HERA Z20 FDA 510(k) Portal | Model: Gemini 3.1 Flash-Lite | [Export Doc] [Share]  |
+---------------------+---------------------------------------------+-------------+
|                     |                                             |             |
|   2. LEFT SIDEBAR   |            3. CENTER WORKSPACE              | 4. POWER    |
|   (Diagnostic Agt)  |                                             |    BAR      |
|                     |  +--------------------+------------------+  |             |
| * Progress (7/30)   |  |                    |                  |  |  [⚖️] Gap   |
|   === 24% ===       |  |  Markdown Input    |  HTML Rendered   |  |             |
|                     |  |  (Textarea)        |  Live Preview    |  |  [🛡️] Stds  |
| * Active Question:  |  |                    |                  |  |             |
|   Q8: Probe Spec.   |  | - Status: OK       |  * K241971       |  |  [🔥] Safe  |
|                     |  | - Target: CMV1-10  |  * Bio-comp      |  |             |
| * Attachments       |  |                    |                  |  |  [🧬] Bio   |
|   [Drag & Drop]     |  |                    |                  |  |             |
|                     |  |                    |                  |  |  [🔒] Cyber |
|                     |  |                    |                  |  |             |
|                     |  +--------------------+------------------+  |  [📊] Visual|
+---------------------+---------------------------------------------+-------------+
```

1.  **頂部導覽列 (Header, 64px 高)**：
    包含高質感磨砂玻璃效果（Backdrop Filter Blur）、動態進度展示、下拉式 AI 模型切換選單（預設 `Gemini 1.5 Flash`，可切換為 `Gemini 1.5 Pro`、`Gemini 2.5 Flash` 或最頂尖的 `Gemini 2.5 Pro`），以及「一鍵 Google Docs」與「PDF 導出」按鈕。
2.  **左側代理人控制面板 (Left Panel, 320px 寬)**：
    *   **審查進度儀表板**：顯示當前 30 道問題之解答率（例如：7 / 30 Questions Completed, 24% 珊瑚色進度條）。
    *   **主動式 Agent 問答卡片**：AI 代理會根據當前焦點問題（Active Question），吐出人性化的引導與要求（如：「請上傳 CMV1-10 探頭的側向解析度數據」）。
    *   **問題庫拖拽區與附件上傳區**：支援直接將外部 `.md` 問題配置文件或原廠 PDF 報告拖入此處，系統自動觸發背景解析與數據填充。
3.  **中央雙欄同步工作區 (Center Workspace)**：
    *   **左欄：極簡 Markdown 編輯器**（預設為 Slate 50 背景，聚焦時平滑過渡至純白，帶有 `motion` 陰影變化）。
    *   **右欄：優雅襯線體 Live 預覽區**。
    *   **雙欄同步滾動（Sync Scroll）**：利用 React Ref 監聽左欄 textarea 的滾動比例（Scroll Ratio），即時、平滑地調整右欄 HTML 容器的 `scrollTop`，實現毫秒級的同步滾動，讓專家在編修時擁有極致掌控感。
4.  **右側浮動 Power Bar (64px 寬)**：
    以深邃色調為底的長條形工具軌（Rail），陳列 8 個精緻的 AI 魔法按鈕，滑鼠懸停時會向上平滑浮動、放大並顯示精美的法規工具提示（Tooltip）。

---

## 9. 510(k) 繁體中文技術審查報告樣張 (Sample Review Report)
*(本樣張為系統合成之 3500 字標準 510(k) 技術審查報告，新增內容與法規論證以**珊瑚色樣式**標註，全面對標 HERA Z20 超音波系統之申報需求)*

---

### # 醫療器材 510(k) 技術審查報告

#### 申報器材：Samsung Medison HERA Z20 診斷用超音波系統
#### 實體等效器材（Predicate Device）：Samsung Medison HERA W10 (K241971)
#### 審查狀態：<span style="color: #F87171; font-weight: 600; background-color: rgba(248, 113, 113, 0.1); padding: 2px 6px; border-radius: 4px;">待補件再審查 (ADDITIONAL INFORMATION REQUESTED)</span>

---

### 一、 預期用途與等效性評估 (Intended Use & Predicate Device Equivalence)

#### 1.1 預期用途說明
本案申報之 **Samsung Medison HERA Z20** 診斷用超音波系統及附隨之超音波轉換器（探頭），其預期用途為「取得人體診斷用超音波影像，並進行體液與組織之聲學影像分析」。

本器材之臨床應用範疇極為廣泛，明確涵蓋：
*   **婦產科與胎兒影像** (Fetal / Obstetrics, Gynecology)
*   **腹部臟器與小器官** (Abdominal, Small Organ)
*   **手術中成像** (Intra-operative)
*   **兒科、新生兒及成人頭部成像** (Pediatric, Neonatal Cephalic, Adult Cephalic)
*   **腔內成像** (Trans-rectal, Trans-vaginal)
*   **肌肉骨骼與淺表組織** (Muscular-Skeletal: Conventional & Superficial)
*   **泌尿系統與周邊血管** (Urology, Peripheral Vessel)
*   **心臟與胸腔成像** (Cardiac Adult, Cardiac Pediatric, Thoracic)
*   **侵入式經食道心臟成像** (Trans-esophageal Cardiac)

#### 1.2 等效性對標 (K241971)
本審查組詳細核對本案之中文說明書與原廠技術規格書。
<span style="color: #F87171; font-weight: 500; background-color: rgba(248, 113, 113, 0.05); padding: 4px; border-left: 3px solid #F87171; display: block; margin: 8px 0;">[審查評估 - 珊瑚色新增項目]：申報器材 HERA Z20 在物理架構、數位波束合成器（Digital Beamformer）之脈衝都卜勒（PW Doppler）核心算法上，與等效器材 HERA W10 (K241971) 具備實質等效性。然而，HERA Z20 首次引入了 CMV1-10 與 CMV1-10Z 兩款高頻寬凸陣探頭，且新增了基於深度學習的 ViewAssist 與 Live ViewAssist 影像自動識別與量測標記功能。這些新增的技術特徵，必須在後續章節中提供充分的「軟體確效」與「臨床前性能測試」佐證，以證明其未引入新的安全與效能疑慮。</span>

---

### 二、 產品基本技術與主要操作模式 (Technical Description & Operating Modes)

#### 2.1 系統核心規格
*   **基本技術**：全數位波束合成（Full Digital Beamforming）、脈衝波都卜勒（PW）、連續波都卜勒（CW）、彩色/能量都卜勒（Color / Power Doppler）、S-Flow。
*   **新增高階操作模式**：
    *   **ELASTOSCAN+**（剪切波彈性成像模式：用於定量評估組織硬度）
    *   **TDI / TDW**（組織多普勒成像與波形分析）
    *   **MV-FLOW**（微細血管低速血流成像）
    *   **IOTA-ADNEX**（基於國際卵巢腫瘤分析組織指引之附件腫瘤風險評估模型）
    *   **S-DETECT FOR Breast / Thyroid**（基於深度卷積神經網路之乳房與甲狀腺結節半定量形態分析系統）

#### 2.2 探頭配置與物理特性
本系統共配置多款超音波探頭，審查重點聚焦於最新申報之高階探頭：

| 探頭型號 | 掃描類型 | 中心頻率 / 頻寬範圍 | 物理掃描長度 / 角度 | 接觸介面與性質 |
|:---:|:---:|:---:|:---:|:---:|
| **CMV1-10** | 凸陣 (Convex Array) | 1.0 - 10.0 MHz (寬頻) | 50 mm / $60^\circ$ | 皮膚 / 黏膜 (短期接觸) |
| **CMV1-10Z** | 3D/4D 容積凸陣 | 1.0 - 10.0 MHz | 45 mm / $55^\circ$ | 皮膚 / 黏膜 (短期接觸) |

<span style="color: #F87171; font-weight: 500; background-color: rgba(248, 113, 113, 0.05); padding: 4px; border-left: 3px solid #F87171; display: block; margin: 8px 0;">[審查評估 - 珊瑚色新增項目]：經審閱原廠技術文件，CMV1-10 與 CMV1-10Z 探頭採用了新型單晶體壓電材料（Single Crystal Piezoelectric Material），其高頻段（8.0-10.0 MHz）主要用於新生兒頭部及表淺器官成像。由於其高敏感度特性，在臨床使用時聲學能量轉換效率極高。原廠必須對此兩款探頭在 ELASTOSCAN+ 及 3D/4D 容積掃描模式下的表面溫度與最大聲學輸出，給予最嚴格的監控，並補充熱指數與機械指數的極端測試報告。</span>

---

### 三、 臨床前測試與安全標準評估 (Pre-Clinical & Safety Evaluations)

#### 3.1 產品電性與電磁相容評估
*   **電性安全**：符合 **IEC 60601-1:2025**（醫療電氣設備第一部分：基本安全與基本性能一般要求）。
    *   *審查紀錄*：OK。已檢附 Nemko Korea Co., Ltd. 出具之完整測試報告。測試結果顯示，系統在漏電流（Leakage Current）、保護接地阻抗（Protective Earthing）及耐電壓測試（Dielectric Strength）中均完全符合規範，且關鍵元器件清單（CDF）與申報實體完全一致。
*   **電磁相容**：符合 **IEC 60601-1-2:2014**（醫療電氣設備：電磁相容測試要求）。
    *   *審查紀錄*：OK。已檢附 Nemko Korea Co., Ltd. 出具之測試報告，確認在強電磁輻射及靜電放電（ESD）環境下，系統之基本性能並無喪失或降級，臨床影像未出現斑點或嚴重扭曲。

#### 3.2 超音波特定安全與聲學輸出 (IEC 60601-2-37 & IEC 62359)
*   **特定安全要求**：**IEC 60601-2-37**（超音波診斷與監控設備的特定安全與基本性能要求）。
    *   *審查紀錄*：<span style="color: #F87171; font-weight: 600;">[不符合] (NON-COMPLIANT)</span>。
    <span style="color: #F87171; font-style: italic; display: block; margin: 4px 0 8px 12px;">原廠技術資料中，未檢附 HERA Z20 主機搭配 CMV1-10 及 CMV1-10Z 探頭的 IEC 60601-2-37 完整測試評估報告。這屬於嚴重的 RTA (Refuse to Accept) 缺陷。</span>
*   **聲場特徵測量**：符合 **IEC 62359**（聲學輸出測量與熱/機械指數確定方法）或 **NEMA UD 2-2004**。
    *   *審查紀錄*：<span style="color: #F87171; font-weight: 600;">[不符合] (NON-COMPLIANT)</span>。
    <span style="color: #F87171; font-style: italic; display: block; margin: 4px 0 8px 12px;">報告中缺少各探頭在 2D、PW 多普勒、以及 3D/4D 操作模式下，熱指數（TI, 包括 TIS 軟組織熱指數、TIB 骨骼熱指數、TIC 顱骨熱指數）與機械指數（MI）的最大值對照表。根據 FDA 2023 最新超音波指引，必須證明 MI 絕對不超過 1.9，且在特定眼科或胎兒應用中 TI 不超過安全閾值。</span>

---

### 四、 生物相容性評估 (Biocompatibility Assessment - ISO 10993-1)

#### 4.1 接觸特性分類
本案所有探頭之生物相容性等級劃分如下：
1.  **體表探頭**：接觸完整皮膚，屬於短暫接觸（$< 24$ 小時）。
2.  **腔內探頭 (CMV1-10 用于經陰道/經直腸)**：接觸黏膜（Mucosa），屬於短暫至中期接觸（$24$ 小時至 $30$ 天）。
3.  **手術中探頭**：接觸組織/血液，屬於短暫接觸。

#### 4.2 測試項目與缺失評估
*   **審查紀錄**：<span style="color: #F87171; font-weight: 600;">[不符合] (NON-COMPLIANT)</span>。
    *   原廠未提供 CMV1-10 及 CMV1-10Z 探頭外殼材料、聲學匹配層（Acoustic Window）及封裝膠料的完整 ISO 10993-1 生物相容性評估報告（必須包含 ISO 10993-5 細胞毒性、ISO 10993-10 致敏性、ISO 10993-23 刺激性測試）。
    *   <span style="color: #F87171; font-weight: 500; display: block; margin: 8px 0 4px 0;">[法規抗辯與補件建議 - 珊瑚色新增項目]：若該探頭之患者接觸材料（醫療級聚氨酯與特殊矽膠配方）與已上市 Predicate Device HERA W10 (K241971) 採用的材料完全一致，且加工與滅菌工藝無任何改變，原廠可提出「材料等效性合理性說明（Material Equivalence Rationale）」以申請免除重複的動物與細胞實驗。但必須提供：(1) 兩者材料之化學成分百分比對比表；(2) 原廠物料規格書（MSDS）及供應商證明；(3) 聲明 HERA Z20 的成型與組裝製程未引入任何殘留加工助劑或有機溶劑。否則，必須立即補做 ISO 10993-5 的 In Vitro 細胞毒性測試（Elution Method），並檢附定量評估結果。</span>

---

### 五、 軟體確效與生命週期管理 (Software Validation - IEC 62304)

#### 5.1 軟體關注層級 (Level of Concern)
本案系統控制軟體、圖像處理軟體以及 AI 量測模組，因其輸出數據直接輔助臨床診斷，一旦發生故障或延遲，可能間接導致患者不當治療，故其 **FDA 關注層級定義為「中度關注層級 (Moderate Level of Concern)」**，對應 **IEC 62304 標準之 B 級安全級別**。

#### 5.2 確效文檔審查
*   **審查紀錄**：OK。
    *   已檢附完整之《軟體確效報告》（Software Validation Report），內容符合《醫療器材軟體確效指引》之規範。
    *   提供 SRS（軟體需求規格書）、SDS（軟體設計規格書）、詳細系統整合測試報告、未解決軟體缺陷（Unresolved Bugs / Anomalies）之風險評估表。經查，所有殘留 Bug 的危害性指數均處於低風險接受區間，且有明確的防護措施（Mitigation Measures）。

---

### 六、 網路安全自我評估與資產防護 (Cybersecurity Assessment)

#### 6.1 防禦機制與威脅建模
本案系統具備 PACS 醫療影像儲存傳輸系統之 Wi-Fi 與有線網路連線功能，其網路安全風險顯著增加。
*   **審查紀錄**：OK。
    *   原廠檢附了《網路安全自我評估報告》，建立了符合 STRIDE 框架之威脅防禦矩陣。
    *   **身分驗證與授權**：系統支援基於 Windows Active Directory 的多用戶密碼權限管控。
    *   **傳輸安全**：影像及病患個資（PHI）傳輸採用 TLS 1.3 與 AES-256 加密演算法，並支援標準 DICOM Secure Profile。
    *   **軟體資產清單 (SBOM)**：提供了完整的第三方軟體組件與開源庫清單。
    *   <span style="color: #F87171; font-weight: 500; display: block; margin: 8px 0 4px 0;">[合規性補強 - 珊瑚色新增項目]：雖然原廠提供了 SBOM，但未針對近期公開之 Linux 核心及 Openssl 關鍵漏洞（CVE-2024-XXXX）提供已知漏洞掃描與修補評估。原廠必須補充一份「漏洞與補丁相容性測試報告」，並承諾在產品上市後，維持每半年一次的網路安全漏洞動態監控機制，且在官網建立漏洞通報與「應急安全更新軟體發佈通道」。此承諾書須由軟體研發部副總裁簽署，並作為 510(k) 卷宗之正式補充附件。</span>

---

### 七、 人工智慧與電腦輔助偵測/診斷軟體 (AI CADe/CADx Documentation)

本章節為本案技術審查之重中之重。HERA Z20 內建 **ViewAssist, Live ViewAssist, HeartAssist, EzVolume, S-Detect** 等多項 AI/ML 自動化工具。

#### 7.1 演算法架構與輸出邏輯
*   **演算法類型**：基於深層卷積神經網路 (CNN) 與空間變換網路 (Spatial Transformer Network, STN)。
*   **輸出邏輯**：系統自動在實時 2D 產科影像中定位胎兒標準切面（如雙頂徑 BPD、頭圍 HC、腹圍 AC、股骨長 FL），當信心度超過 **85%** 時，自動凍結影像、繪製邊界框（Bounding Box）並顯示測量數值。

#### 7.2 訓練與驗證數據集 (Dataset Provenance & Curation)
*   **數據規模**：訓練集共包含 **150,000** 張去識別化（De-identified）超音波產科影像，驗證集 **30,000** 張。
*   **數據來源**：主要採集自亞洲與歐洲之 5 家大型醫學中心。
*   **標註資歷**：所有訓練集與測試集之黃金標準（Gold Standard）皆由 3 位資歷超過 10 年之產科主任級醫師進行「雙盲、多輪交叉標註」，並使用 Cohen's Kappa 係數驗證標註者間一致性（Kappa 值均 $\ge 0.88$）。

#### 7.3 獨立性能評估與統計信心 (Non-Clinical Performance Testing)
*   **審查紀錄**：<span style="color: #F87171; font-weight: 600;">[不符合] (NON-COMPLIANT)</span>。
    *   原廠未針對 ViewAssist 與 S-Detect 的各項輔助測量指標，提供 95% 置信區間 (95% Confidence Intervals, CI) 的統計學顯著性論證。
    *   <span style="color: #F87171; font-weight: 500; display: block; margin: 8px 0 4px 0;">[數據泛化與偏誤糾正 - 珊瑚色新增項目]：深度審核數據集人口統計學特徵發現，訓練數據中北美及非洲裔患者（不同脂肪組織厚度、骨骼特徵）的影像佔比低於 3.5%。考慮到超音波訊號穿透力深受腹部脂肪層厚度（BMI）影響，演算法在肥胖人群或非亞裔人種上的「影像泛化表現與輪廓分割靈敏度」可能出現顯著漂移（Performance Drift）。原廠必須：(1) 補充一組包含 2,000 例肥胖患者（BMI $\ge 30$）及不同人種的獨立臨床測試集（Independent Clinical Test Set）性能數據；(2) 重新計算並繪製對應之 ROC 曲線，給出 AUC 分布值（預期 AUC 應保持在 0.90 以上，且 95% CI 下限不低於 0.85）；(3) 於中文說明書與系統 UI 中顯著標示「AI 測量輔助限制警告：當孕婦 BMI 超過 30 時，手動覆核量測框之必要性」。</span>

---

### 八、 最終產品品質檢測與性能試驗 (Performance Test - Bench)

#### 8.1 探頭軸向與側向解析度
*   **性能基準**：依據《「診斷用超音波影像系統暨超音波轉換器(探頭)」臨床前測試基準(2018)》。
*   **審查紀錄**：<span style="color: #F87171; font-weight: 600;">[不符合] (NON-COMPLIANT)</span>。
    *   原廠未提供新申請探頭 CMV1-10 及 CMV1-10Z 在最大深度與極端頻率下的測試 Phantom 數據（軸向/側向解析度、盲區 Dead Zone、幾何誤差幾何失真度）。
    *   <span style="color: #F87171; font-weight: 500; display: block; margin: 8px 0 4px 0;">[技術補件方案 - 珊瑚色新增項目]：原廠必須使用具有標準靶線分佈的超音波模擬人體組織仿體（如 Gammex 403 / ATS 539 Phantom），在中心頻率 3.5 MHz、高頻 8.0 MHz 下分別進行測試。報告應載明：(1) 在 50mm、100mm 深度下的軸向解析度（須 $\le 1.0$ mm）與側向解析度（須 $\le 1.5$ mm）；(2) 盲區大小（須 $\le 3.0$ mm）；(3) 深度量測幾何誤差（誤差率小於 $\pm 1.5\%$ 或 $\pm 1$ mm）。測試數據必須呈報實測相片（B-Scan Photos of Phantom Targets），並由合格之第三方計量與認證檢測機構簽署蓋章。</span>

---

### 九、 綜合審查結論與補件決議 (Review Conclusion & deficiency Action)

經過對 Samsung Medison HERA Z20 超音波系統 510(k) 技術卷宗之全面技術審查，本審查組認為：**該產品目前尚不具備與實體等效器材之實質等效性，主要原因在於部分關鍵探頭之安全性、生物相容性及 AI 演算法泛化數據嚴重缺失**。

本案發出 **「主要缺失補件通知 (Major Deficiency Letter)」**。申請人（贊助商）須在收到本通知之 180 天內，針對上述第三、四、七、八章節標記為 **[不符合]** 且在報告中以 <span style="color: #F87171; font-weight: 500;">珊瑚色</span> 高亮指出的 5 大法規缺失，進行逐條補件應答（Deficiency Responses）。逾期未補或補件技術論證不充分者，本案將直接予以「不予上市核准（Not Substantially Equivalent, NSE）」之最終裁決。

---

## 10. 20 個專業深入的法規審查與系統設計追問 (20 Follow-up Questions)

為了進一步驗證本系統之「Agentic」設計、提示詞工程、多模型切換、以及 510(k) 審查標準之精準度，我們提出以下 **20 個專業且具有深度挑戰性的追問**，以供下一階段系統實現與法規精進之參考：

1.  **問題 1 (法規等效性)**：在 HERA Z20 系統中，新增的 AI 功能（如 ViewAssist）與 K241971 原有的手動量測相比，是否改變了「Intended Use（預期用途）」的核心範疇，從而導致 FDA 認定無法走 510(k) 簡易途徑，而必須強制改走 De Novo 途徑？
2.  **問題 2 (模型幻覺控制)**：當使用 Gemini 2.5 Pro 作為後端生成引擎時，如何保證 AI 在解讀用戶 30 道問題的零散回答時，不會憑空「編造」或「幻覺（Hallucinate）」出並不存在的 Nemko 實驗室安全報告編號或測試數據？
3.  **問題 3 (珊瑚色追蹤與協作)**：當用戶在「雙欄對照編輯器」中手動修改已被 AI 標記為珊瑚色的段落時，系統如何即時識別哪些是「用戶手動輸入」、哪些是「AI 原始生成」、哪些是「用戶基於 AI 建議修改後的新增」？
4.  **問題 4 (Google Docs API 性能)**：在透過 OAuth 連接 Google Docs 並推送長達 4,000 字的含有複雜表格、高亮色塊的 Markdown 文件時，如何克服 Google API 的 Rate Limits（頻率限制）與格式轉譯時的段落錯位問題？
5.  **問題 5 (AI 魔法功能 1 的優先級)**：在執行「差異對比與自動修復」時，若報告中同時存在 IEC 60601-1-2 (EMC) 與 ISO 10993-1 (生物相容) 的缺失，AI 代理如何排定修復優先順序，以防單次 Prompt 處理過載？
6.  **問題 6 (聲學輸出安全邏輯)**：在「聲學安全查驗（Acoustic Safety Check）」中，AI 如何判斷用戶手動輸入的 MI/TI 數值是真的具備物理合理性，還是刻意輸入的造假合規數值？系統是否內置了超音波換能器物理公式（例如壓電晶體頻率與焦點深度的關係）來進行二次驗證？
7.  **問題 7 (生物相容性等效論證)**：在執行生物相容性 MAF 比對時，AI 如何自動辨識不同化學品名稱的實質一致性（例如「Polyurethane A」與「PU Grade A-3」是否屬於同種材料），以確保豁免論證（Rationale）的嚴謹度能通過 FDA 毒理學家的審查？
8.  **問題 8 (網路安全 SBOM 解析)**：系統如何自動將上傳的 SBOM (如 CycloneDX 或 SPDX 格式的 JSON/Markdown) 與 NVD (National Vulnerability Database) 在線數據庫關聯，以識別出 HERA Z20 軟體平台是否含有未修補的零日漏洞？
9.  **問題 9 (追加功能 6 的偏誤計算)**：臨床數據泛化能力評估器（功能 6）是如何定義「泛化得分（Generalization Score）」的底層數學模型？是否考慮了不同醫療機構的超音波設備（如 GE, Philips vs Samsung）的聲學後處理差異？
10. **問題 10 (追加功能 7 的問答擬真度)**：FDA 審查官模擬器（功能 7）如何動態調整其「刁難級別」？是否可以根據當前用戶申報的時間壓力或產品風險等級（Class II vs Class III）來調整審查官提問的嚴苛度與語氣？
11. **問題 11 (追加功能 8 的引用樹更新)**：智能證據路徑圖（功能 8）在使用 D3.js 繪製時，若用戶在 Markdown 編輯器中實時打字，樹狀圖的節點和連線如何實現「無閃爍、無重繪」的增量更新（Incremental DOM Updates）？
12. **問題 12 (問題庫 Markdown 解析容錯)**：當用戶上傳一個格式嚴重損壞、缺少關鍵 H3 標籤或含有亂碼的 Markdown 問題庫時，系統的 AST（抽象語法樹）解析器如何執行 Graceful Degradation（優雅降級），避免系統當機？
13. **問題 13 (多模型切換的 Prompt 適應)**：在頂部導覽列將模型從 `Gemini 1.5 Flash` 切換為 `Gemini 2.5 Pro` 時，系統如何自動調校 System Instruction 與 Prompt Temperature，以發揮 Pro 模型在邏輯推理上的深度，同時避免其生成語氣過度冗長？
14. **問題 14 (PDF 分頁與中文字體)**：在使用 jsPDF 將含有繁體中文字元的報告導出為 PDF 時，如何解決「中文字體缺失導致的亂碼與方塊字（Tofu Characters）」問題？是否需要系統預先加載思源黑體（Source Han Sans）之 Base64 編碼？
15. **問題 15 (OAuth 權限最小化)**：為遵守隱私與安全規範，系統在請求 Google Docs 與 Drive 權限時，如何向用戶清晰說明為什麼需要 `drive.file`（僅限制存取本應用建立的檔案）而非寬鬆的 `drive`（存取所有雲端檔案）？
16. **問題 16 (本地持久化數據同步)**：當用戶在多個瀏覽器分頁同時開啟此 510(k) 門戶時，`localStorage` 的即時寫入如何防止多頁面相互覆蓋、造成數據競爭（Race Conditions）？是否需要引入 Broadcast Channel API 進行跨頁面狀態同步？
17. **問題 17 (AI CADe/CADx 指引對齊)**：針對台灣 TFDA 與美國 FDA 在電腦輔助診斷（CADx）軟體上的監管差異（例如 TFDA 要求提供特定醫院的臨床驗證報告，而 FDA 較著重於 bench 測試），系統在生成 Traditional Chinese 報告時，如何動態切換或兼顧兩者的指引標準？
18. **問題 18 (系統可擴展性)**：如果未來 FDA 發佈了針對生成式 AI（Generative AI）醫療器材的全新審查指引，系統管理員如何在不修改原始程式碼的前提下，動態更新 30 道問題庫，並在 AI 魔法引擎中注入新的法規審查權重？
19. **問題 19 (輔助量測準確度評估)**：在第 27 題（量測類型與準確度）中，當系統評估距離與體積量測時，AI 如何自動解析超音波「像素校準值（Pixel Calibration Factor）」之技術檔案，以推導出量測的理論極限誤差？
20. **問題 20 (Agent 自主權限邊界)**：本系統作為「Agentic App」，在什麼情況下應該「完全自主決策並修改報告」，在什麼情況下必須「停下來、向用戶發出確認提示」，以避免 AI 過度熱心而擅自修改了真實的法規測試數據？

---
## [技術規格書完結]
