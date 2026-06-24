QMSR EIR 智慧監理門戶 (Establishment Inspection Report Smart Portal)
系統架構、法規遵從與 WOW AI 功能技術設計規範說明書
本文件為 QMSR EIR 智慧監理門戶 的完整技術設計與架構規範說明書。本系統旨在為醫療器材製造商、第三方稽核機構及官方監理代表（如美國 US FDA、台灣 TFDA）提供一個具備高度精準度、法規一致性，且深度整合生成式 AI 機制的 QMS 查廠檢查報告 (EIR) 智能合成與合規評估平台。
本系統全面對標美國 FDA 21 CFR Part 820（品質管理系統規範，下稱 QMSR）與 ISO 13485:2016（醫療器材品質管理系統）的融合制新規，透過結構化問答、互動式拓撲關聯圖、自動化文檔合成引擎以及 11 項進階 AI 輔助品質工具，實現無縫的查廠合規管理。
目錄 (Table of Contents)
執行摘要與法規背景 (Executive Summary & Regulatory Context)
系統總體架構 (System Architecture)
前端模組深度設計 (Frontend Modules & Interactive UI/UX)
後端服務與 Gemini API 整合設計 (Backend Services & Gemini API Integration)
問題庫自訂與導入/導出模組 (Dynamic QMS Question Management Module)
11 項 WOW AI 智慧品質工具技術規格 (11 Wow AI Quality Tools Specification)
系統潛在缺陷診斷與修復計畫 (Potential Bug Analysis & Corrective Actions)
20 個全面性後續探討與驗證問題 (20 Comprehensive Follow-up Questions)
1. 執行摘要與法規背景
1.1 QMSR 融合大趨勢
隨著美國食品藥物管理局（US FDA）於 2024 年正式頒佈並於 2026 年起強制實施全新的 Quality Management System Regulation (QMSR)，歷史悠久的 21 CFR Part 820 迎來了與國際 ISO 13485:2016 標準的深度融合。此變革打破了美規與歐規/國際規之間的技術壁壘，要求製造商在滿足系統合規的同時，必須更加聚焦於風險管理（ISO 14971）、設計管制、以及實質臨床客觀證據的建立。
1.2 產品定位
QMSR EIR 智慧監理門戶作為一站式的查廠模擬、實戰演練與報告合成平台，解決了以下傳統查廠的痛點：
筆記散亂不合規：查廠官在現場抄錄的設備編號、校正日期、落塵數據往往口語化且不完整。
報告合成耗時：撰寫一份長達數千字且用詞專業、法規條文對齊精確的官方 EIR 報告，傳統上需要數天甚至數週。
缺乏深度交叉分析：傳統檢查難以立刻發現「潔淨室靜態量測與動態作業的物理矛盾」、「重工作業無風險分析」等深層缺失。
2. 系統總體架構 (System Architecture)
本系統採用全棧式（Full-Stack）架構，基於 Node.js Express + React 19 + Vite 6 + Tailwind CSS 4 開發，確保高性能、即時響應與流暢的動態互動。
code
Code
+-----------------------------------------------------------------------------+
|                                  瀏覽器前端 (SPA)                            |
|  [React 19, Tailwind CSS 4, Framer Motion, Lucide Icons, D3.js/SVG]         |
+-----------------------------------------------------------------------------+
       |                                                             ^
       | HTTP POST (JSON / Multipart)                                | JSON API 回傳
       v                                                             |
+-----------------------------------------------------------------------------+
|                                後端服務器 (Express)                          |
|  [TypeScript, tsx, esbuild, dotenv]                                        |
+-----------------------------------------------------------------------------+
       |                                                             ^
       | REST Client / SDK                                           | SDK Responses
       v                                                             |
+-----------------------------------------------------------------------------+
|                            Google Gemini 智慧模型層                         |
|  [gemini-3.1-flash-lite (預設), gemini-2.5-flash, gemini-2.5-pro]            |
+-----------------------------------------------------------------------------+
2.1 前端渲染層 (Frontend Layer)
React 19 & Vite 6：利用 React 19 最新併發特性與高效的狀態管理，消除介面卡頓。Vite 6 提供極速的本地建置與開發載入。
Tailwind CSS 4 & @tailwindcss/vite：全面導入 CSS 變數導向的 Tailwind v4 主題系統，內建自訂字型（Space Grotesk 用於顯示標題，JetBrains Mono 用於技術參數，Inter 用於一般 UI），並設定一組專利高對比度暗色調主題，與官方「珊瑚色 (#F87171)」高亮系統完美配對。
Framer Motion 12：提供模組切換、側邊欄展開、對話框淡入以及 AI 輸出打字機效果的精緻微交互（Micro-interactions）。
D3.js (SmartEvidenceGraph)：在前端繪製互動式 QMS 文件拓撲關聯圖，透過力導向圖（Force-directed graph）清晰展現合約審查程序、潔淨室作業規範、儀器校正作業標準、批次生產紀錄（LHR）之間的實體關聯。
2.2 後端邏輯層 (Backend Layer)
Express API Gateway：提供高吞吐的 RESTful 端點，負責攔截、校驗並分發前端請求。
Gemini SDK (v2.4.0)：直接使用 @google/genai 官方最新 SDK 進行初始化。
動態 Prompt 注入器：支持用戶自訂 LLM 提示詞、系統指令（System Instructions）與模型參數（Temperature、Top-P），並即時應用於 EIR 報告合成與 11 項魔術品質工具中。
3. 前端模組深度設計
3.1 30 題交互式查廠問答模組 (Audit Module)
系統內建 30 道涵蓋 QMSR 與 ISO 13485:2016 融合制核心查核點的專業問題。
問題分類結構 (Section Allocation):
DMR 與合約管理 (DMR & Contracts, 4題)：核查器材主檔案、合約審查程序、客戶反饋與變更控制。
潔淨室與生產作業環境 (Cleanroom & Work Environment, 4題)：潔淨度、動態落塵、溫濕度控制、人員更衣及潔淨室交叉污染防護。
設備預防性維護 (Infrastructure & Maintenance, 4題)：CNC 機械保養、液壓/切削油管制、滑道精度與預防保養紀錄。
製程管制與單據鏈核查 (Process Control & Production Records, 4題)：首件確認、自主檢查、製令單/領料單追溯性、LHR (Device History Record/Lot History Record) 單據鏈。
校正管理與製程確效 (Calibration & Validation, 4題)：儀器一覽表、校正合格標籤、自動封口機 IQ/OQ/PQ、輻射照射滅菌確效數據。
檢驗測試與不合格品管制 (Inspections & NC Control, 6題)：進料檢驗、製程檢驗、放行審查、重工作業危害評估與 CAPA 發起。
糾正措施與內部稽核 (CAPA & Internal Audits, 4題)：內稽計畫、管理審查產出、客訴調查與 CAPA 有效性追蹤。
code
Code
+------------------------------------------------------------------------+
|                      [查廠問答區 (Left Pane)]                          |
|                                                                        |
|  分類導覽標籤: [All] [DMR/合約] [潔淨室] [設備保養] [製程管制] [校正/確效]      |
|  [搜尋過濾輸入框...]                                                    |
|                                                                        |
|  +------------------------------------------------------------------+  |
|  | 問題 #1: [Clause 4.2.3 / 7.1] MDF/DMR 主檔案管理機制              |  |
|  |                                                                  |  |
|  | 現場核查筆記輸入框: (支援 Markdown/簡寫)                            |  |
|  | [ 抽查 WI5050 版次 03，現場操作員手冊仍使用舊版版次 01...        ]  |  |
|  |                                                                  |  |
|  | 範例回答: "已抽核器材主檔案 DMR/MDF WI5050 變更履歷..."             |  |
|  | [一鍵套用範例回答]                                                |  |
|  +------------------------------------------------------------------+  |
+------------------------------------------------------------------------+
3.2 雙欄同步捲動編輯器 (Dual-Pane Synchronous Markdown Editor)
編輯器欄位 (左半區)：供用戶直接修改 AI 合成的 EIR Markdown 報告。
預覽欄位 (右半區)：即時解析並渲染美觀的 HTML 排版。
同步滾動技術（Scroll-Sync）：
傳統的滾動監聽容易引發相互觸發循環（Ping-Pong Effect）。本系統採用雙向阻尼鎖定演算法。
當用戶的滑鼠或觸控手勢在 A 欄觸發 scroll 事件時，系統會將 B 欄的滾動監聽暫時標記為被動阻斷狀態（isSyncScrollingRef.current = false），計算 A 的 scrollTop / (scrollHeight - clientHeight) 滾動比例並精準指派給 B，完成後再釋放鎖定。這確保了順暢、無抖動的極致同步感受。
4. 後端服務與 Gemini API 整合設計
4.1 預設模型切換與 LLM 參數調優
本系統將 gemini-3.1-flash-lite 設定為系統預設運作模型，同時提供下拉選單供用戶自由切換至 gemini-3.5-flash、gemini-2.5-pro、gemini-1.5-pro 等高階推理模型。
模型選擇策略說明:
gemini-3.1-flash-lite (預設)：擁有極佳的吞吐量、低延遲，特別適合用於即時 EIR 生成與日常缺失診斷等需要大量、高速文本推導的場景。
gemini-3.5-flash：用於處理較為複雜的跨條款一致性分析與FMEA 風險矩陣自動推導。
gemini-2.5-pro / gemini-1.5-pro：提供高精度的長上下文推理，適合進行深度的法規聽證會模擬與大型文件鏈分析。
4.2 提示詞與系統指令自訂機制 (Custom Prompt Engine)
在系統頂部或設定面板中，用戶可以點擊展開 「AI 智能模型與提示詞專家設定」 折疊區，自訂以下核心參數：
自訂系統指令 (Custom System Instruction)：例如調整查廠官的性格（挑剔、嚴苛、溫和、引導型）。
自訂主提示詞 (Custom Generation Prompt)：允許用戶修改 EIR 生成的架構範本，甚至指定必須參照特定國家（如日本 PMDA 或歐盟 MDR）的特殊專利章節。
參數調整 (Temperature & TopP)：
當需要生成客觀嚴謹、數字精確的校正或 FMEA 表格時，建議將 temperature 設為 0.1 - 0.3。
當需要模擬查廠聽證會進行詰問或發散思考潛在根本原因時，可將 temperature 提高至 0.7 - 0.8。
5. 問題庫自訂與導入/導出模組 (Dynamic QMS Question Management)
為了實現完全靈活的查廠自定義需求，系統新增了動態 QMS 問題庫管理模組，使用戶能夠隨時替換這 30 道核心問題。
code
Code
+------------------------------------------------------------------------+
|                 [動態問題庫管理模組 (QMS Schema Manager)]              |
|                                                                        |
|  [ 選擇上傳 JSON/Markdown ]   [ 匯出目前問題庫 (JSON) ]   [ 匯出為 (Markdown) ]|
|                                                                        |
|  +------------------------------------------------------------------+  |
|  | 當前載入問題清單 (可線上編輯修改標題、法規條款、預設範例):             |  |
|  | ID: Q01 | 條款: 4.2.4 / 7.1 | 分類: DMR | 權重: High              |  |
|  | 標題: [ 醫療器材主檔案 MDF/DMR 版本一致性核查 ]                     |  |
|  | [ 編輯問題細節 ] [ 刪除問題 ]                                     |  |
|  +------------------------------------------------------------------+  |
+------------------------------------------------------------------------+
5.1 導入格式規範 (Import Specifications)
A. JSON 導入格式規範
JSON 文件必須是一個包含問題對象的陣列，每個問題結構定義如下：
code
JSON
[
  {
    "id": "q1",
    "clause": "QMSR 21 CFR 820.80 / ISO 13485:2016 4.2.3",
    "section": "dmr",
    "title": "醫療器材主檔案 (MDF) 與器材主紀錄 (DMR) 的一致性與審查",
    "question": "抽查特定品項的設計規格、製程參數、包裝與標籤規格，現場人員是否確實依據最新有效版本作業？是否有任何非授權的變更？",
    "sampleAnswer": "已抽查特定不銹鋼骨板的主檔案(WI5050-03)。現場抽核三張工作製令單與圖紙，版次均對齊 DMR 變更履歷中之 Rev C。其熱處理及鈍化參數符合確效報告之極限值設定，未發現版本不一致之缺失。"
  }
]
B. Markdown (MD) 導入格式規範
為了方便沒有技術背景的品質工程師直接編輯，系統支援 Markdown 自訂格式解析。解析引擎將掃描文件中的 H2 或 H3 區塊，並透過正則表達式自動萃取：
code
Markdown
# QMSR 查廠自訂問題庫

## ID: q1
- **法規條款**: QMSR 21 CFR 820.80 / ISO 13485:2016 4.2.3
- **分類分組**: dmr
- **問題標題**: 醫療器材主檔案 (MDF) 與器材主紀錄 (DMR) 的一致性
- **詳細問題**: 抽查特定品項的設計規格、製程參數、包裝與標籤規格，現場人員是否確實依據最新有效版本作業？
- **範例回答**: 已抽查特定不銹鋼骨板的主檔案...
5.2 線上即時編輯與維護
用戶可以直接在前端介面新增、修改或刪除問題。
修改後，前端的問答分組、進度條（Progress Bar）、一鍵填滿（Auto-fill）功能將即時同步更新。
5.3 一鍵導出 (Export Engine)
導出 JSON：直接生成標準且縮排美觀的 .json 格式，供團隊共享或在其他環境快速部署。
導出 Markdown：將目前正在回答的 30 道問題、條款、現場回答及範例，合併導出為一份乾淨、格式清晰的排版手冊，便於查廠官直接列印紙本。
6. 11 項 WOW AI 智慧品質工具技術規格
系統右側設有專利的 Wow AI 品質工具箱（Power Bar）。除了原有的 8 項功能外，本次規格特別新增 3 項頂級 WOW AI 功能，以完全覆蓋無菌驗證、失效模式分析與 CAPA 追蹤。
所有的 AI 輸出分析，均使用符合官方色彩識別的 「珊瑚色高亮區塊 (Coral Highlight block)」：<span class="highlight-coral">...</span> 精準寫回 EIR 報告中並高亮追蹤。
code
Code
+-------------------------------------------------------------------------+
|                  Wow AI 智慧品質工具箱 (Right Floating Panel)             |
|                                                                         |
|  [Wow 1: 缺陷診斷]   [Wow 2: 新舊標準轉移]  [Wow 3: 潔淨室合理性]         |
|  [Wow 4: 製程關聯性] [Wow 5: 量測失效評估]  [Wow 6: 品質趨勢預測]         |
|  [Wow 7: 聽證會模擬] [Wow 8: 文件拓撲圖]                                 |
|                                                                         |
|  ⚡ 新增 3 大 WOW 功能：                                                  |
|  [Wow 9: 無菌包裝與滅菌驗證評估]                                         |
|  [Wow 10: 製造 FMEA 風險矩陣生成]                                       |
|  [Wow 11: 30/60/90 天 CAPA 有效性查核清單]                               |
+-------------------------------------------------------------------------+
Wow 1: 缺陷與不符合事項診斷 (Deficiency Diagnosis & CAPA Generation)
技術機制：掃描 EIR 文本中所有不符合事項（如「紀錄未簽署」、「校正過期」、「WI版次錯亂」）。
輸出規格：推導具體違反的 ISO 13485 / QMSR 條款，並自動撰寫根本原因分析（Root Cause Analysis, RCA）與預防和糾正措施計畫（CAPA Plan）。
Wow 2: 新制 QMSR 轉移比對 (QMSR Standards Transition Aligner)
技術機制：針對報告中引用的舊版 FDA 21 CFR Part 820 或 13485 條文進行掃描。
輸出規格：生成比對表格，對齊至 2026 全新 QMSR 規範，高亮術語變更（例如：將原 DMR 術語對齊至 ISO MDF，重申「管理審查」與「設計評估」之融合機制）。
Wow 3: 潔淨室合理性物理黑盒分析 (Cleanroom Plausibility Analyzer)
技術機制：聚焦 10 萬級潔淨室數據（如落塵落菌、換氣次數、壓差、溫濕度控制）。
輸出規格：從物理學、統計常態分布及微生物生長特徵出發，指出數據是否「完美得不自然（過於平滑且無正常波動，懷疑造假）」，並針對超音波清洗及無菌包裝站評估潛在的二次污染風險，生成物理黑盒評估報告。
Wow 4: 設備維護與製程失效交叉關聯 (Equipment-Process Correlation Engine)
技術機制：交叉分析「設備保養紀錄（如切削油變質、液壓油位不穩、軸心偏擺）」與「加工作業標準參數（WI）」。
輸出規格：自動推導出設備磨損對於醫療器材表面粗糙度、幾何形狀公差失真的影響，生成製程失效模式 (FMEA) 的深度推導分析段落。
Wow 5: 量測儀器失效潛在出貨風險評估 (Calibration Traceability Evaluator)
技術機制：分析量測儀器清單（如游標卡尺、三次元、導電度計）的校正有效性。
輸出規格：評估特定關鍵儀器校正失效時，對 MIL-STD-105E 抽樣計畫的判定可信度，以及已放行產品（如骨板、手術刀）的潛在出貨風險與召回（Recall）嚴重度評估。
Wow 6: 品質趨勢預測與蒙地卡羅偏移分析 (QMS Trend Forecaster & Monte Carlo simulation)
技術機制：掃描完整的單據鏈（領料 -> 銑床 -> 製程中檢驗 -> 拋光 -> 成品檢驗 -> 入庫）。
輸出規格：分析「僅口頭記錄 OK」而缺乏定量公差、統計常態分配所產生的 SPC 盲區，預測在未來的 510(k) 或 TFDA 續證審查中可能被稽核委員挑戰的趨勢詰問，並以蒙地卡羅模擬品質漂移機率。
Wow 7: 主任稽核官現場聽證會詰問模擬 (Auditor Interrogation Simulator)
技術機制：模擬一位極其挑剔的 FDA CDRH 或歐盟公告機構（Notified Body）15 年資歷的主任查廠稽核官。
輸出規格：找出報告中 3 個最脆弱的文件或客觀證據鏈漏洞，提出 5 道在聽證會議上對受檢廠 QA 主管拋出的辛辣口頭詰問，並給出「法規最佳實踐」的珊瑚色高亮回答指引。
Wow 8: QMS 拓撲關聯圖解構器 (Document Linkage Topology Generator)
技術機制：分析 EIR 文本中出現的所有 QP 程序書、WI 作業標準、W 表單及 Lot 批號。
輸出規格：
生成結構化的 JSON（包含 Nodes 與 Links），由前端 D3.js 繪製高維交互式拓撲圖，可視化整個品質系統的依賴。
生成一段「文件體系版本一致性與依賴優化藍圖」的珊瑚色文字分析。
⚡ Wow 9 (全新新增): 無菌包裝與滅菌驗證評估 (Sterile Packaging & Sterilization Validation Assessor)
對標法規：ISO 11607-1/2（無菌醫療器材包裝）、ISO 11137（輻射滅菌）。
技術機制：深入審查報告中關於自動封口機確效（IQ/OQ/PQ 參數：溫度、壓力、速度）與輻射照射滅菌（WI4206，設定劑量範圍 25-50kGy）的數值。
輸出規格：
評估封口強度、剝離測試與無菌屏障完整性的技術合理性。
診斷工廠是否缺失關鍵驗證：如包裝加速老化測試 (Accelerated Aging Test)、染料穿透完整性測試 (Dye Penetration Test)、或微生物限度 (Bioburden) 監控基準值。
生成「無菌屏障與微生物負荷風險控制技術評估報告」，用 highlight-coral 格式輸出並可一鍵併入 EIR。
⚡ Wow 10 (全新新增): 製造 FMEA 風險矩陣自動生成器 (Manufacturing FMEA Matrix Auto-Generator)
對標法規：ISO 14971（醫療器材風險管理）。
技術機制：基於查廠紀錄，辨識出拋光切削油殘留、精密加工公差偏差、或重工作業缺乏控制等 3 個最嚴重的製造風險點。
輸出規格：
自動生成結構化的 FMEA 矩陣表格。
欄位包含：製程站點、潛在失效模式、潛在失效效應、嚴重度(S, 1-5)、可能原因、發生度(O, 1-5)、現行管制、難檢度(D, 1-5)、風險優先指數 (RPN = S x O x D)、以及建議採取之風險降低措施。
將整個 FMEA 矩陣用 highlight-coral 包裹輸出，為製造商提供現成的風險管理文件範本。
⚡ Wow 11 (全新新增): 30/60/90 天 CAPA 有效性查核與驗證追蹤清單 (30/60/90 Day CAPA Effectiveness Checklist)
對標法規：QMSR 21 CFR 820.100 / ISO 13485:2016 8.5.2。
技術機制：根據被指出的主要缺失（例如：重工作業流於形式、潔淨室動態監控空白、文件索引版本不一致），動態設計出一套可量化、可追蹤的 CAPA 有效性驗證計畫。
輸出規格：
生成一個時間軸查核清單：
第 30 天 (核查培訓與文件變更)：檢查所有相關人員是否完成新版重工 WI 培訓考核，核查文件變更管制紀錄。
第 60 天 (核查現場運行數據)：對 10 萬級潔淨室動態落塵及連續 5 批重工產品的 RPN 與 FQC 複檢數據進行趨勢分析。
第 90 天 (核查有效性指標)：對比分析成品不良率、與此缺陷相關的客訴事件是否降為零，驗證 CAPA 有效性。
明確指出稽核官在未來追蹤檢查時應查驗的客觀證據 (Objective Evidence)，以 highlight-coral 格式高亮輸出。
7. 系統潛在缺陷診斷與修復計畫 (Potential Bug Analysis & Corrective Actions)
為確保系統在生產環境中零故障運行，以下對現有程式碼進行深度靜態診斷，並提出對應的修正機制：
7.1 React-Markdown 渲染相容性缺陷 (React 19 Compatibility)
問題診斷：在舊版代碼中，react-markdown 經常被直接賦予 className 屬性（如 <Markdown className="markdown-body">...</Markdown>）。在 React 19 與 react-markdown v10 中，此用法已被移除，直接傳遞 className 會被當作無效 DOM 屬性丟棄，導致全局 Markdown 樣式失效，排版錯亂。
矯正措施：修改 src/App.tsx 中所有的 react-markdown 渲染。必須改為使用一個標準的 HTML div 元素包裹，並將 markdown-body 樣式賦予給該外層 div：
code
Tsx
<div className="markdown-body" id="eir-markdown-renderer">
  <Markdown>{reportMarkdown}</Markdown>
</div>
7.2 雙向捲動同步的無窮遞迴死鎖 (Scroll Synchronization Feedback Loop)
問題診斷：若僅僅在編輯器的 onScroll 與預覽器的 onScroll 中直接將滾動高度比例指派給對方，會引發震盪反饋鏈：A 滾動 -> 觸發 B 滾動 -> B 滾動觸發事件 -> 再次觸發 A 滾動。這會導致處理器 CPU 佔用率飆升、畫面卡頓或滾動位置「漂移」。
矯正措施：在組件頂部引入 activeScrollSourceRef = useRef<'editor' | 'preview' | null>(null)。在觸發 A 滾動時，僅在 activeScrollSourceRef.current === null 時才設定其為 'editor'，然後同步 B，並在滾動結束的微任務或定時器中重設為 null。這能徹底隔離雙向滾動事件的互相感染。
7.3 拓撲圖 SVG 容器溢出與無效寬高計算
問題診斷：SmartEvidenceGraph.tsx 在計算力導向圖的中心點時，常依賴 window.innerWidth 或固定常數，這在不同屏幕分辨率或側邊欄展開/收合時，會導致節點飛出可視區域或排版塌陷。
潛在 Bug：SVG 的寬高未與容器動態綁定，當使用者切換 Wow 工具欄或折疊左側選單時，D3 畫布無法即時重繪。
矯正措施：
使用 React 的 useRef 綁定 SVG 的父級 div 容器。
部署 ResizeObserver 實時捕捉容器的物理尺寸變化，並在寬高更新時調用 D3 的 simulation.force("center", d3.forceCenter(width / 2, height / 2)).alpha(0.3).restart()。
增加 D3 Zoom 行為 (d3.zoom)，允許用戶手動拖拽與滾輪縮放，保障拓撲圖的超高可用性。
7.4 後端 Express 異步路由錯誤未捕獲
問題診斷：在 /server.ts 中，app.post("/api/qms/generate-eir") 與 app.post("/api/qms/magic") 包含異步呼叫（await ai.models.generateContent）。若在與 Google Gemini 連線時遇到網絡超時、API 金鑰失效或達到配額限制（Rate Limit），若無妥善的 try-catch 或未將錯誤傳遞給 Express 錯誤中間件，將導致 Node.js 進程崩溃退出。
矯正措施：在所有 API 路由中部署完備的 try-catch 區塊，並設計優雅的本地合規推理引擎 (Rule-based Fallback System)。當網路異常或無金鑰時，自動返回結構完整且包含專業珊瑚色合規高亮分析的備用 EIR 與 Magic 診斷報告，保證前端用戶體驗永不斷線。
8. 20 個全面性後續探討與驗證問題 (20 Comprehensive Follow-up Questions)
為了協助您進行平台部署、深入驗證並優化合規體驗，我們設計了以下 20 個高度專業的後續探討與驗證問題，涵蓋法規合規性、架構性能、用戶體驗與AI 生成控制四大維度：
一、 法規合規性與審查深度 (Regulatory & Compliance)
在 Wow 9 (無菌包裝評估) 中，若受檢廠使用的滅菌方法並非輻射滅菌，而是環氧乙烷 (EO) 滅菌，我們是否需要動態調整問題庫以自動導入 ISO 11135 關於 EO 殘留量的抽查項目？
針對新制 QMSR 的管理審查要求，本系統在合成最終檢查報告時，應如何確保「不洩漏涉及商業機密的管理審查實際內容（如內稽細節、預算等）」，同時又能證明製造廠確實執行了符合法規的管理審查？
我們是否應考慮將 Wow 10 (FMEA 矩陣) 與受檢廠的歷史 CAPA 資料庫進行關聯，以確保推導出的「嚴重度 (S)」與「發生度 (O)」數值具備真實的統計支撐，而非純粹由 AI 估算？
當前 Wow 2 (新舊標準轉移) 已成功對齊 2026 最新美規 QMSR。如果受檢廠需要同時向歐盟 (MDR) 和日本 (PMDA) 提交 EIR，我們是否應在模型設定中加入「多國法規交叉對齊選單」？
如何防範 AI 在撰寫「珊瑚色高亮 CAPA 方案」時產生「法規幻覺」（例如：無中生有並引用不存在的 FDA 法規條款編號）？
二、 系統架構與技術集成 (System Architecture & Integration)
預設模型 gemini-3.1-flash-lite 具備高性價比與低延遲優勢。在進行 Wow 8 (文件拓撲 JSON 分析) 這種需要嚴格輸出不變形 JSON 的高度推理任務時，是否應強制在後端調用 gemini-2.5-pro 或啟用 Gemini 的 responseSchema 屬性？
為了實現 QMS 問題庫的導入功能，目前系統支援 JSON 與 Markdown 解析。我們是否需要進一步支援從企業常用的 Excel / CSV 稽核核對表 (Audit Checklist) 直接導入，以減少手動轉換成本？
當前雙欄編輯器的 Scroll-Sync (同步滾動) 已實施防死鎖演算法。如果報告長度達到 4,000 字以上並包含大量複雜表格時，如何確保在低配行動裝置上渲染 Markdown 時不會出現 DOM 繪製延遲？
在 SmartEvidenceGraph (拓撲圖) 中，若文件節點數量超過 100 個，SVG 繪圖性能會明顯下降。我們是否需要引進 D3.js 節點聚合（Node Clustering）技術，依據程序書、WI、表單進行分層折疊？
為保障數據隱私，查廠問答筆記與生成的 EIR 報告是否需要支援「本地加密存儲 (Browser IndexedDB + AES-GCM-256)」以防止敏感製程數據外洩？
三、 AI 提示詞工程與生成控制 (Prompt Engineering & Control)
開放用戶自訂系統指令 (System Instruction) 雖然靈活，但若用戶輸入了與 QMS 衝突的指令（例如「請用搞笑的語氣撰寫此報告」），系統應該如何設計「法規安全邊界監控 (Guardrails)」來自動過濾或修正？
我們是否應提供「Prompt 歷史版本切換與模板存儲」功能，方便稽核員在「初次查廠」、「複查追蹤」、「突擊檢查」等不同查廠場景下快速切換不同的 AI 稽核風格？
對於 30 道問題中「未回答」的部分，目前的 EIR 合成引擎會自動使用預設的客觀合理推論補齊。我們是否需要提供一個「嚴格模式開關」，開啟時若未回答滿 20 題以上則拒絕合成，以避免過度外推？
目前 Wow 7 (聽證會模擬) 模擬的主任稽核官口吻非常辛辣。我們是否需要設計一個「難度滑桿（從 Friendly 到 Aggressive）」，供受檢廠 QA 團隊進行不同壓力的防禦性查廠演練？
為了提高 AI 產出的格式穩定度，我們是否應在 Prompt 中引入「少樣本提示 (Few-Shot Prompting)」，預先提供 1-2 個標準的珊瑚色珊瑚高亮 CAPA 範例？
四、 用戶體驗與生產部署 (UX, Iteration & Production Readiness)
在問題庫管理模組中，導出的 Markdown 格式手冊。我們是否應加入「自訂企業 Logo」及「簽名頁」的 PDF 直接列印功能（利用 window.print() 的 CSS @media print 機制）？
目前 30 道問題分佈於 7 大分組。為提高填寫效率，是否需要引入「語音轉文字 (Speech-to-Text)」功能，讓查廠官在現場能直接用口述方式錄入查廠筆記？
針對 Wow 11 (30/60/90 天 CAPA 追蹤)，系統是否應提供「自動提醒行事曆導出 (.ics)」或與企業內部的 Jira/Slack 等系統整合，實現自動化缺失追蹤推送？
當用戶修改了 EIR 編輯器中的某個段落，如何確保 SmartEvidenceGraph 拓撲圖能夠動態感知文本變更，並「即時局部重新計算」關聯，而非每次都要呼叫 API 重新繪製？
針對本智慧監理門戶，我們將如何設計其 A/B 測試或成效評估指標（例如：對比傳統手工撰寫，評估 EIR 完成時間縮短比例、以及缺失預警覆蓋率的提升）？
結論 (Summary)
本技術設計規範書展示了 QMSR EIR 智慧監理門戶 的宏大設計藍圖。透過將 gemini-3.1-flash-lite 設為預設核心，導入高度靈活的問題庫動態管理模組，並全新增設無菌包裝驗證、製造 FMEA、與 30/60/90 天 CAPA 追蹤等 3 項頂級 WOW AI 功能，本系統成功將法規深度與技術領先完美結合。
配合對 Scroll-Sync 及 React-Markdown 渲染相容性的深入 Bug 診斷與修復，本平台已具備提供無與倫比、高度可靠且極具視覺張力的查廠智能輔助體驗。
