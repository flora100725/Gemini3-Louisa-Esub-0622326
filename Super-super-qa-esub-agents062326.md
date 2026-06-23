HERA Z20 510(k) 智能輔助審評平台：系統架構與技術規格說明書
<span style="color: #ff6f61">System Architecture & Technical Specification Document</span>
文件版本： v3.0
發佈日期： 2026 年 06 月 23 日
文件性質： 系統設計與高階技術白皮書（Confidential - Technical Specification）
目標產品： HERA Z20 510(k) 臨床前審查報告、AI 查驗登記與補件落差決策工具
主題視覺色圈： 珊瑚粉（Coral #ff6f61）、深沉岩石灰（Slate Dark #0f172a）、柔和乳白（Off-white #f8fafc）
<span style="color: #ff6f61">一、 專案概述與核心設計理念 (Project Overview & Philosophy)</span>
1.1 研發背景與法規痛點
在醫療器材產業中，申請美國食品藥物管理局（US FDA）之 510(k) 上市許可或台灣衛生福利部食品藥物管理署（TFDA）之查驗登記，向來是一項高難度、高成本且需要跨學門知識整合的龐大工程。
以三星麥迪遜（Samsung Medison）的診斷用超音波系統「HERA Z20」為例，雖然該主機在美國已順利獲得多型號授權大案（案號：K241971，包絡 HERA Z20、R20、HERA Z30 及 R30）之核准，但是在將其引進台灣進行單一型號（HERA Z20）新申請登記的過程中，往往會遭遇以下法規痛點：
多型號包絡與單一型號申請之差距證明： 如何有效向審查官證明單一機型 HERA Z20 的硬體規格（如數位波束合成器、聲功率輸出控制等）完全被母案 K241971 的安全包絡所涵蓋，而不會引入全新安全性與有效性問題。
多項關鍵資料不符合（Non-Compliant）： 原廠報告在超音波探頭特殊安全要求（IEC 60601-2-37）、探頭生物相容性完整分析（ISO 10993-1）、臨床量測精度規格、以及最具爭議的人工智慧／機器學習（AI/ML）電腦輔助診斷（CADe/CADx，如 ViewAssist 與 HeartAssist）臨床科學驗證卷宗存在明顯資料缺失，極易遭受退件（Refuse to Accept, RTA）。
高難度技術卷宗撰寫負擔： 補件信（Deficiency Letter）之回覆需要極強的法規官腔術語、嚴謹的工程學幻影（Phantom）物理實驗設計、以及模型漂移（Model Drift）上市後維護計畫，耗費法規專員（RA）數月之精力。
1.2 平台核心定位
本平台旨在打造一款「AI 驅動型醫療器材監理智囊合議艙」，專為台灣總代理商、原廠研發主管與內部法規科學家優化 HERA Z20 510(k) 審查與補件工作。
平台跳脫常規 AI 應用的「空洞對談罐頭機制」，藉由 30 題標準法規 stepper、8 大深度硬核 regulatory AI magic 分析工作流，將冰冷的原始測試回覆，一鍵轉譯成高達 3500 至 4000 字、結構完美且排版優雅的 珊瑚粉視覺主題（Coral Color Theme） 專業級 510(k) 技術審查報告（Technical Review Report）。
1.3 視覺與交互設計規範
本系統嚴格遵循「專業、純淨、去冗餘（Anti-AI-Slop）」的極簡高階設計美學：
色彩搭配（Theme Pairings）： 採用大面積深沉 Slate-900 作為工作艙背景，搭配 #f8fafc 的高易讀文字。系統一級、二級標題與重大警告框採用醒目的珊瑚色（Coral #ff6f61），其在暗色底色下呈現出卓越的視覺對比。
字型指引（Typography）： 標題採用高張力、科技感強的 Inter 搭配無襯線系統字體，涉及技術參數、代碼與 JSON 模型定義時則自動調用 JetBrains Mono 等寬字體。
動畫微交互（Interaction & Motion）： 全盤使用 motion（motion/react）控制視窗與 Stepper 卡片，包括滑動淡入（fade-and-slide）、漸層填充進度條等，避免機械性生硬切換。
誠實與人性化標籤（Aesthetic Sincerity）： 嚴禁在頁面邊角堆砌無意義的網路 ping 值、硬體虛擬日誌、PORT:3000 或 "CORE_SYSTEM_ONLINE" 等噱頭裝飾。所有 UI 控制鍵皆使用最樸實、精確的醫學工程標籤，將視覺焦點完全保留在 510(k) 報告本文的精準論述上。
<span style="color: #ff6f61">二、 全面保留之既有功能規格規格解析 (Original Features Review)</span>
本系統在升級設計中，完全保留並優化了既有的四大核心功能，確保使用者工作流的穩定延續。
2.1 30 題分步審查問卷狀態引擎 (30-Question Stepper Questionnaire Engine)
系統建立了一個全面涵蓋 510(k) 及 TFDA 診斷用超音波查驗要點的 30 題狀態化問卷。
狀態追蹤機制： 答案由 Record<number, string> 的 React 狀態進行全局維護，並在瀏覽器本地（LocalStorage）實現毫秒級自動同步快照，防止輸入流失。
指標分類架構：
Q1~Q2: 基礎行政與比對背景（K241971 大案與 HERA Z20 單一型號對接）
Q3~Q4: 電性安全與電磁相容性（IEC 60601-1:2025、IEC 60601-1-2:2024 新舊版本差距）
Q5~Q6: 超音波特殊安全標準（IEC 60601-2-37 缺失補件進度、TI / MI 聲發射）
Q7: 探頭型號與規格比對（CMV1-10, CMV1-10Z 等 12 款探頭參數）
Q8~Q9: 生物相容性評估（ISO 10993-1 細胞毒性、致敏與黏膜刺激 GLP 測試缺漏補件）
Q10~Q11: 軟體確效工作（IEC 62304 生命週期、關切等級、異常追蹤（Bug Tracking））
Q12: 醫療器材網路安全（威脅建模（Threat Modeling）、DICOM 傳輸金鑰與漏洞防衛）
Q13~Q19: AI/ML 電腦輔助功能之 CADe/CADx 指引深水區（演算法特徵、數據集多樣性、獨立盲測集隔離度、性能指標 95% 信心區間、模型漂移（Model Drift）定期校準計畫）
Q20~Q23: 性能測試與量測準確性（軸向／側向解析度、距離與流速精度測試合格標準、 finished QC report）
Q24~Q25: 與既有獲准器材比對（數位波束合成、對照 Predicate 機型比對包絡）
Q26~Q30: 聲學安全與補件時程計畫（ALARA 控制機制、TFDA 基準表、補件時限、 9 個月承諾期）
AI 單題自動填寫機制（Single-Q Autofill）： 面對高難度技術缺失問題，使用者隨時可點擊單題填寫。系統背景將該問題的所有法規引導 placeholder 與類別封裝，經由後端代理向 Gemini 高階推理伺服器提出精巧 prompt，返回 150 - 300 字專業、合規、用語嚴密且吻合原廠 K241971 背景的 Traditional Chinese 答覆。
AI 一鍵全題填補機制（Fast-Track Fill-All）： 針對需要快速生成全套測試樣板或進行合規演示的用戶，提供一鍵智能編配。10 秒內為 30 個問題灌注極高水準的安全測試虛擬數據與承諾比對性答案，快速填為綠色「已答」狀態。
2.2 專業級 AI 智慧報告工作室與珊瑚色 Markdown 實時渲染器 (AI Report Studio)
問卷遞交後，後端 Express 的 /api/generate-report 路由將調用高階 Gemini 模型。
長度與結構規範： 強制模型吐出 3000 至 4000 字、橫跨十一大核心章節的極致詳盡報告。
珊瑚色強視覺特徵： Markdown 渲染器經過專門調正。所有 Markdown 中的一、二級標題，以及報告本文中的 <div style="border-left: 4px solid #ff6f61;"> 與強警告文字，會嚴格轉接為主導性的珊瑚粉色（#ff6f61）。
雙模式預覽與編輯模組（Live Markdown Preview & Editor）： 使用者可在「實時渲染預覽（Preview）」和「原始 Markdown 編輯器（Edit）」之間隨意切換。編輯器模式下支援實時輸入、手動擴寫或修改，渲染視窗在 50 毫秒內更新其珊瑚粉排版外觀，提供極度流暢的即時反饋。
即時對話式報告微調（Session Refinement）： 底部附設 AI 微調對話框（Refinement Chat Console）。使用者輸入例如「請將第七章關於 CADe 盲測資料集的多中心描述額外充實五百字」或「幫我在報告最後追加一段關於探頭消毒連續耐熱試驗的專家意見」，系統將原報告和該指令發送至後端並返回重繪後的優質 Markdown 內容，保持珊瑚粉重點標記不丟失。
2.3 8 大監理魔法模組分析工作流 (8 Regulatory Magic Workflows)
系統左下方或專屬分頁裝配了八個「一鍵震撼（WOW）」的 AI Regulatory Workflows。
工作流 ID	工作流外觀名稱 (Taiwan Terminology)	法規深度分析要點 (Analysis Focus)
gap_analysis	法規缺陷與補件落差分析	核實 IEC 60601-2-37 與 ISO 10993-1 中最隱蔽被退件的缺失，精確繪製 Gap 對照表。
se_predictor	實質等同性（SE）核准概率精算	計算 HERA Z20 在台美核准率，拆解硬體、演算法與臨床三大等同度百分比。
audit_drafter	自動缺失官方回函撰寫 (Response Letter)	以三星Medison代理人名義，撰寫官腔標準、氣勢與學術法規邏輯無瑕之回覆補件信。
biocompatibility_protocol	探頭生物相容性確效計畫書	針對接觸人體皮膚與陰道黏膜之 CMV1-10 探頭，設計必測項目的 GLP 試驗流程。
cybersecurity_threat	網路安全威脅矩陣評估	剖析 DICOM 與 PACS 潛在 5 大漏洞，建立吻合最新指引的安全阻絕與追蹤矩陣。
ai_software_discloser	AI影像軟體演算法深度揭露技術卷宗	揭露 ViewAssist U-Net 架構、百萬級標註一致性 Kappa 值與 Model Drift 校準策略。
measurement_verifier	臨床量測精度實體確效設計	規範距離、心功能與波峰流速在音波仿真實體假體（Phantom）上的臨床前確效方案。
predicate_harmonizer	經典對比器材規格調和判定	調和 Z20 與 K241971 母案多型號之聲發射包絡，申辯規格並無引入全新安全性危慮。
分析成果融合機制（Merge to Dossier）： 點擊任何魔法工作流並經 AI 深度算力編織生成後，其長達 2000 字的專題分析成果會呈現於魔法面板，使用者一鍵點擊「向下融合至主報告（Add to Bottom）」即可自動將該魔法專案追補到主報告之中，迅速擴大報告文字量與硬實力。
2.4 多格式本地匯出與題庫生命週期維護 (Export & Customize)
高品質 Markdown 檔案下載： 直接下載 .md 的 UTF-8 純文字，利於 RA 在本機 git 保存。
高相容性 HTML 檔案導出： 自動封裝內嵌 CSS（包含精心挑選的 Inter 網頁字型與珊瑚粉標籤類（class）語義），可在任何設備上雙擊完美閱覽。
MS Word 兼容式 .doc 格式封裝： 透過封裝 Microsoft Word XML 命名空間與 Word A4 邊距（Margins），導出的檔案在 Word 及 Google Docs 中直接開啟，保留完整的 Table 邊線與珊瑚粉重點樣式。
高保真 A4 安全邊距 PDF 列印： 通過 @media print 專屬 CSS，移除導航與無關側邊欄（no-print），格式化 A4 列印排版學，使用家在點擊「PDF 匯出」時調出系統列印介面直接列印或儲存為 PDF，不產生溢出或半行截斷。
自定義 Markdown 檢核題庫引擎： 本地端開放自定義題庫上傳。使用者將自己的 Markdown 文件拖放於此並進行在線即時編輯，解析器會語義識別數字清單，直接覆蓋並加載成最新的 Question 30-Stepper，徹底解放題庫格式限制。
<span style="color: #ff6f61">三、 三大新增之「WOW」旗艦智慧功能技術建構與交互設計 (3 WOW Features Specification)</span>
為了讓本平台從「實用級合規工具」躍升至「世界頂尖的探針式智能監理系統」，我們在此高階說明書中特別規劃並設計了以下三個全新的 Wow 旗艦功能。這三個功能完全在原有的高對比深色 Slate-Coral 視覺地圖下進行模組化部署，無需破壞原有的功能底座。
3.1 WOW 功能 A：動態多維度補件成熟度指針與核准機率評級引擎 (Dynamic Clearance Maturity Radar & AI Scoring Engine)
核心技術概念：
在醫材查驗登記過程中，最致命的不是「回答不全」，而是「不知道目前的回答是否達到了主管機關（TFDA/FDA）的及格安全水位」。
本功能在平台問卷區的上端部署一個 動態補件成熟度指針系統，其內部由一組「語義法規特徵計分演算法」構成，針對用戶已填寫的 30 題答案進行動態掃描。
演算法將 30 題劃分為 5 大核心合規維度，若檢測到特定維度存在不符合（Non-Compliant）字樣，或文字密度不足，計分將進行權重折抵：
維度一：電性器具安全性（Electrical & Hardware Safety）[
]
一項未通過或未檢附 IEC 60601-1 / IEC 60601-2-37 判定，該維度計分歸零。
維度二：人體生物安全評鑑（Biocompatibility Risk）[
]
評估 ISO 10993 是否明確提報細胞致敏測試合格，未填即扣除滿分。
維度三：AI CADe 臨床科學有效性（AI/ML Software Clinical Validity）[
]
檢查數據集多樣性、獨立盲測集、信心區間與 Model Drift 維護。
維度四：臨床性能精度確效（Acoustic & Performance Accuracy）[
]
考核超音波解析度、距離與 Doppler 幻影誤差是否有提出數字比對。
維度五：比對背景一致性與行政完整性（Regulatory Harmony）[
]
評測說明書是否合規、TFDA 對照表填寫進度。
由後端 Gemini 微調模型針對 30 題總回答，進行「主管機關一次性過關概率總評級（Overall Clearance Rating）」，分為：
D (Dismissal / RTA 風險極高)： 總得分 
。警告：安規、生物相容缺漏，一旦送件有 95% 概率在第一週被退件。
C (Deficiency / 預期重大補件)： 總得分 
。預估審查委員將下發高達 10 項以上的 Deficiency 補件，建議立即使用 AI Magic 生成 B-Protocol 或 CADe Discloser。
B (Substantially Equivalent / 快道補件水位)： 總得分 
。等同性完整，預估僅有 2~3 項程序性補件。
A (Clearance Approved / 一次通關評級)： 總得分 
。數據細節均勻，物理確效具備，極高概率直接放行。
在問卷頁頂部呈現一個精美的「多維度合規雷達動態環（Dynamic Compliance Ring）」。利用 SVG 曲線，結合 motion 動態繪製。環的外圈顏色隨得分動態漸變：從低分時的「深灰至高亮珊瑚粉紅 #ff6f61」到高分時的「珊瑚粉漸變至極光金 #f1c40f」。
大面積警告條（Maturity Alert Panel）： 若計分低於 70%，面板下方自動刷出一個由雙重珊瑚色粗實線（border-left-width: 4px; border-color: #ff6f61;）包裹的「阻退警告（RTA Watchdog）」卡片，列出當前計分被扣分的致命 blocker 題目（例如：未在第 8 題回答探頭 GLP 級刺激測試報告編號，或未在第 15 題交代 ViewAssist 測試用臨床醫院名單），引導用戶一鍵跳轉定位填補。
code
Code
+---------------------------------------------------------------------------------+
| [Maturity: 62% - DEFICIENCY LEVEL]                      [ Clearance: 48.5% ]     |
| (SVG Compliance Radar Arc: Progressing from Charcoal with Coral Accents #ff6f61)|
|                                                                                 |
|  ! Critical Blocker: Q5 (IEC 60601-2-37 Report) has no physical verification.    |
|  ! Critical Blocker: Q13 (AI Software Dataset Size) lacks statistical 95% CI.   |
|  [--> Jump to Fix Q5]  [--> Autofill All Blockers via Regulatory AI]            |
+---------------------------------------------------------------------------------+
3.2 WOW 功能 B：多源合規文檔 PDF/Word 語義特徵提取與問卷自適應填充套件 (Smart Multi-Source Dossier Extractor)
核心技術概念：
在真實的專案現場，RA 專員手上其實擁有許多零散的英文文件、原廠 datasheet
或 Nemko 電控安全證書摘要，但要手動將這些英文規格複製、翻譯並分門別類填寫進平台的 30 個 Stepper 清單中，耗時費神。
本功能在題庫維護區，特別注入了 多源合規文檔智能拖放特徵提取器。
跨格式解析管線（Document Parsing Pipeline）： 前端利用 File API 將用戶上傳的文檔（.pdf、.docx、.txt、.md）轉為純文字位元（ArrayBuffer），如果是大文件，則提取前 50KB 之純文字核心技術段落以防 token 溢出。
法規命名實體特徵識別（Regulatory Named Entity Recognition, R-NER）：
將讀出的非結構化文本發送至後端的專有 R-NER 語義引擎，專門尋找並擷取超音波的核心合規物理特徵、數值或證號：
安規證號： Nemko Korea, IEC 60601-1, NK-25-E098231 (Matches Q3)
探頭型號： CMV1-10, CMV1-10Z, Center freq 4.5MHz (Matches Q7)
生物相容： Cytotoxicity ISO 10993-5, Grade 0, Non-sensitizer (Matches Q8)
AI指標： ViewAssist, Receiver Operating Characteristic, AUC 0.976 (Matches Q18)
聲學強度： Mechanical Index, MI 1.9, Ispta.3 720 mW/cm2 (Matches Q6)
計算提取出的英/德/韓文技术片段，與平台 30 個 Stepper 題目語義相似度（Cosine Similarity On Vector Embeddings）：
將相似度 
 的第一候選題目刷出，直接將該片段智能翻譯為 繁體中文專業監理申覆文件，自動填入對應的暫存答案框中。
在「題庫範本維護（Templates）」主面板旁配屬一個具有高辨識度的「拖拽感應框（Dossier Dropzone）」。邊框採用珊瑚粉色的點狀虛線（border-style: dashed; border-color: #ff6f61;），並帶有一條流動的發光粒子特效（利用 Tailwind 的 animate-pulse 與柔和發光陰影 coral-shadow）。
用戶將 Nemko 安全證書 PDF 拖入 Dropzone 後，後端會在 2 秒內完成掃描，彈出一個高對比暗色覆蓋面板，高亮顯示：
「語義提取分析成功！偵測到符合 [Q3 電安規標準] 與 [Q7 探頭型號規格] 技術細節。」
提供一鍵「自適應填入問卷暫存存檔（Merge into Stepper Answers）」按鈕，前端將以 staggered 動態，滑動更新 Stepper 中的答案，並將指示燈轉為綠色「已導入」樣式，使繁瑣的拷貝整理過程，昇華成極客般的智能自動化體驗。
code
Code
+---------------------------------------------------------------------------------+
|  | | | Drag & Drop Reference Dossier (PDF/Word/Markdown) here | | |          |
|  (Dashed Coral Border with Soft Pulse Shadow Glow: #ff6f61)                     |
|                                                                                 |
|  [Scanning...] Found Nemko Certificate "NK-25-E098231" and Probe List.          |
|  * Semantic similarity mapped:                                                   |
|    - Snippet 1 (Nemko Safety)   ===> 95% cos similarity with Q3 (IEC 60601-1)     |
|    - Snippet 2 (Probe Freq)     ===> 88% cos similarity with Q7 (Probe List)     |
|  [Confirm & Populate Stepper]  [Cancel]                                         |
+---------------------------------------------------------------------------------+
3.3 WOW 功能 C：openFDA 數據庫實時關聯與 K-Number 臨床對照包絡比對儀 (FDA Predicate Pack Harmonizer & Cross-Referencer)
核心技術概念：
在 510(k) 申報中，最核心、最折磨研發團隊的，就是「實質等同性比對（Substantial Equivalence）」。你需要將你新申請的產品與歷史上已經獲准的對照器材（Predicate Device）進行地毯式的性能參數對比。
本功能在 AI 監理魔法核心中，特製了 openFDA 全球數據庫比對儀。
當 RA 專員在系統中輸入對照器材的核准案號（K-Number，例如 K241971、K181818）時，系統後端將直接調用 openFDA 官方註冊 API：
https://api.fda.gov/device/510k.json?search=k_number:"K241971"
並即時解析出官方原始 JSON 資料中的核心鍵值：
applicant: 製造商資訊（SAMSUNG MEDISON CO., LTD.）
device_name: 登記器材名稱
indications_for_use: 官方核准適應症原文（此段文字即是 Q2 適用人族群的核心法規對比標竿）
advisory_committee: 審查審評委員會（Obstetrics/Gynecology）
summary_file_url: 該大案原始 510(k) Summary 官方 PDF 文件連結
一般超音波最常被退件的理由，是新申請型號的探頭聲出力（Acoustic Intensity Ispta.3）超出了 Predicate 的核准包絡極限。
本調和分析模組會實時比對：
系統將提取 HERA Z20 各探頭（如 CMV1-10 在多普勒下的物理聲功率強度，與 K241971 中宣告的 720 
 安全極限值），如果候選機型大於 1.0，自動響應高風險預警。
在「AI 監理魔法」頁面頂端配置一個 K-Number 搜索檢索欄。當用戶輸入 K241971 並點擊調和比對後，系統將實時渲染出一個高維、極具設計感的三維實質等同性「等同性參數對比艙（Substantial Equivalence Comparison Grid）」。
左右分立排版，左側展示「HERA Z20 申請型號 (Candidate)」，右側展示「K241971 美國獲准母案 (Predicate)」。
物理規格變異度色塊（Delta Matrix）： 採用高對比珊瑚粉色字體高亮標出兩者的物理規格變異（例如：探頭 CMV1-10 頻率完全重疊（等同）、AI ViewAssist 演算法框架版本一致（等同）、但診斷用說明書中文譯本在『侵入式陰道探頭』的字句上存在 14% 的語義寬度落差）。
儀表盤下方一鍵附帶「生成對照等同性技術宣誓聲明（Generate SE Statement）」按鈕，快速產出 1500 字、完美吻合美台雙邊法規等同、用珊瑚粉色粗體排版的高階 SE 技術申辯證言。
code
Code
+---------------------------------------------------------------------------------+
|  Input Predicate K-Number: [ K241971 ]   [ Run openFDA AI Harmonization Scan ]  |
|                                                                                 |
|                  SUBSTANTIAL EQUIVALENCE (SE) MATRIX ANALYSIS                   |
|                                                                                 |
|       CANDIDATE (HERA Z20 Taiwan)      |      PREDICATE (K241971 FDA US)        |
|  -------------------------------------  |  -----------------------------------  |
|  Acoustic Limit: Ispta.3 710 mW/cm2     |  Acoustic Limit: Ispta.3 720 mW/cm2   |
|  (98.6% Matching - NO NEW SAFETY RISK)  |  (Official FDA Safe Intensity Cap)    |
|                                         |                                       |
|  AI Engine Version: v2.5 (ViewAssist)   |  AI Engine Version: v2.4 (ViewAssist)  |
|  (100% Backwards Compatible - EQUAL)    |  (Certified Diagnostic AI Model)       |
|                                                                                 |
|  [Generate Regulatory Compliance Oath]  [Download FDA Official 510(k) Summary]  |
+---------------------------------------------------------------------------------+
<span style="color: #ff6f61">四、 系統技術架構與數據模型安全防線 (Technical Specification)</span>
本系統雖然是客製化的法規合議解決方案，但在技術棧部署與數據模型上均採用工業級的嚴格標準。
4.1 全棧開發技術棧
前端框架 (Frontend Stack)：
React 18 配合 TypeScript 5 組件開發。
Vite 6 作為極速建置工具（關閉熱模組替換 HMR 以保障在 preview iframe
裡的渲染流暢，防止閃爍）。
全盤採用 Tailwind CSS 4 撰寫極具設計感的 UI 原子類（class）。
使用 motion 庫（motion/react）建立 Stepper 切換時的 Fluid transition 運動軌跡，帶給用戶溫潤、細緻的交互反饋。
後端架構 (Backend Stack)：
使用 Node.js + tsx 驅動 Express 為基礎的多媒體 API 伺服器。
CJS Bundling 關鍵安全防線：
因應 Cloud Run Container 的冷啟動與 Node 原生對 TypeScript ES Module 的嚴格相對路徑檢查阻礙，本專案的生產環境打包腳本設定為：
vite build && esbuild server.ts --bundle --platform=node --format=cjs --packages=external --sourcemap --outfile=dist/server.cjs
藉由 esbuild 將後端的所有 TypeScript 在編譯期 bundle 成單一規格的 CommonJS。這完全繞過了 Node 的多文件 ESM 加載限制，大幅降低了冷啟動延遲並增加了運作穩定度。
高階 AI 推理引擎 (AI Inference)：
採用 Google 官方最新、功能最強大的 @google/genai TypeScript SDK。
安全性限制防護（Lazy Initialization）：
系統在初始化模組時，對 GEMINI_API_KEY 進行了環境變數的安全守護與延遲載入（Lazy Initialization），絕不在全域或模組載入階段就進行強拉（例如 new GoogleGenAI(process.env.GEMINI_API_KEY!)），防止因金鑰未配置造成整個 Node Dev 伺服器崩潰、網頁無限期白屏的問題。
如果金鑰未找到，API 路由在被使用者執行動作時才會拋出格式完美的合規缺漏錯誤，並主導在 UI 呈現清晰配置引導，大幅提升部署彈性。
4.2 後端 API 規格模型設計 (API Routes Schema)
1. 報告編配生成路由 (Generate Report)
Endpoint: POST /api/generate-report
Request Payload (JSON):
code
JSON
{
  "answers": {
    "1": "主機型號 HERA Z20 完全繼承自 K241971，不超出其核准範疇...",
    "2": "中文說明書草案適用對象為胎兒、產科及心臟、周邊微細血管，禁忌症明確...",
    "3": "已檢附由 Nemko Korea 於 2025 出具之 IEC 60601-1 報告，核准編號..."
    // ... up to 30 keys
  },
  "model": "gemini-3.5-flash",
  "customPrompt": "請在此段格外強調探頭 CMV1-10 物理性能在本地臨床試用的重要性"
}
Response Payload (JSON):
code
JSON
{
  "success": true,
  "report": "# <span style=\"color: #ff6f61\">510(k) 臨床前技術審查報告</span>\n\n## <span style=\"color: #ff6f61\">一、 產品基本行政資訊與背景概述</span>..."
}
2. 單題 AI 語義智能填充路由 (Autofill Question)
Endpoint: POST /api/autofill-question
Request Payload (JSON):
code
JSON
{
  "id": 8,
  "question": "本案所有擬申請之接觸人體超音波探頭，是否有檢附依據 ISO 10993-1 之生物相容性評估報告？",
  "category": "生物相容性評估",
  "model": "gemini-3.5-flash"
}
Response Payload (JSON):
code
JSON
{
  "success": true,
  "answer": "關於探頭生物相容性，本公司所擬申請之 CMV1-10 外殼材料、介電透鏡，其配方製程完全等同已獲准之 K241971 器材，無毒理殘留。目前原廠正於 GLP 認證實驗室執行致敏與黏膜刺激重測，預計於補件期限內補齊並提交符合 ISO 10993-1 之合格評估計畫書與確效報告。"
}
3. 8 大監理魔法核心處理路由 (AI Magic Execution)
Endpoint: POST /api/ai-magic
Request Payload (JSON):
code
JSON
{
  "workflowId": "cybersecurity_threat",
  "report": "# <span style=\"color: #ff6f61\">510(k) 臨床前技術審查報告</span>...",
  "answers": { "1": "HERA Z20 規格..." },
  "model": "gemini-3.5-flash"
}
Response Payload (JSON):
code
JSON
{
  "success": true,
  "title": "Cybersecurity Threat Vector Scanner (網路安全威脅矩陣評估)",
  "result": "# <span style=\"color: #ff6f61\">醫療器材安全威脅矩陣評估</span>\n\n### <span style=\"color: #ff6f61\">3.1 五大潛在漏洞攔阻</span>..."
}
<span style="color: #ff6f61">五、 結語與展望 (Summary & Vision)</span>
醫療器材安全關乎大眾生命，其法規查驗與性能確效的技術門檻理應高懸。然而，昂貴而低效的「人工手動撰寫差」與「文檔版本混亂」卻成了阻礙新興高端醫療器材引進、延誤臨床診斷時機的桎梏。
HERA Z20 510(k) 智能輔助審評平台 通過 30-Stepper、8 大監理魔法演算法與三大即將引入之 WOW 旗艦智慧模組，完美打通了「原廠技術底座、主管審查標準及大語言模型學術實力」的三大關隘。
珊瑚色的精準標記與深岩色的沉浸式操作艙，帶給法規科學家與研發高管最精緻、最誠實的高對比尊榮感官體驗。透過這份系統架構與技術規格說明書，我們不僅定義了平台目前的卓越現狀，更為高階監理 AI 的未來發展梳理出一條極具工程細節與法規智慧的必經之路。
<span style="color: #ff6f61">六、 20 大深度專家級隨訪追問議題 (20 Refined Follow-up Questions)</span>
為了在此高階說明書後繼續推動未來的迭代研討並加深與法規高管、審評委員的技術合議，智能助理特別為本平台後續的架構精進、安全性申辯與演算法訓練梳理出以下 20 大專家隨訪追問議題：
6.1 平台技術架構與 UI 交互演進 (Platform UX & Performance)
問題一 [文檔提取極限]： 對於上傳的 20MB 超大原廠英文探頭材料 PDF，如果單次文字提取超出了 Gemini 的 context window 長度或請求字數上限，本系統應如何實施「局部滾動式特徵窗口抽取（Dynamic Chunking & MapReduce）」來避免 token 截斷？
問題二 [成熟度演算法優化]： 補件成熟度指針計分引擎中，當用戶透過 Markdown 範本修改了 questions 題庫數量（例如增加到了 50 題），各題目的權重分數應如何進行「動態權重複歸一自適應比對（Acoustic-Weight Recalibration）」？
問題三 [PDF 網頁字型高保真]： 在 triggerPrintPdf (PDF列印匯出) 過程中，我們如何配置 CSS，使網頁上的 Inter 與 JetBrains Mono 字型，在無 internet 離線狀態下強制打包為 embedded base64 格式，來完全防止轉出的 PDF 出現明體或亂碼現象？
問題四 [ openFDA 在線錯誤降級]： 實施 openFDA API 在線對比時，若適逢 FDA 聯邦伺服器停機維修（HTTP 503 錯誤），我們的前端如何無縫啟動「內嵌 K241971 自主離線基準預載包（Fallback Offline Package）」，來避免阻礙當前會話？
6.2 電安規與超音波暴露指標申辯 (Hardware, Acoustic & Safety)
問題五 [安規舊報告等同論證]： 針對 IEC 60601-1-2:2014 的 EMC 舊檢驗報告，我們如何在 AI 魔法中建立無懈可擊的論述，向台灣 TFDA 證明本系統外殼的「電磁抗擾度金屬屏障隔離技術技術變更（Shielding Verification）」已等同最新 2024 版本安全界線？
問題六 [MI/TI 極限安全裕度]： 在超音波模式切換聲學保護上，HERA Z20 探頭內部溫度可能因連續高負載彩色彩色多普勒而上升。AI 應如何編制探頭內部「熱敏電阻感知切斷電路（Thermal Cut-Off Loop）」之冗餘安全確效宣告？
問題七 [ALARA 臨床最低聲能證明]： 審查委员常追問「如何確保在婦產科成像中嚴格履行 ALARA 臨床最低聲壓限制」。我們應提供何種主機作業系統的「自動發射功率降級（Auto acoustic degradation）節能鎖」軟體驗證證明？
問題八 [寬頻探頭通道等同物理]： 新款 CMV1-10 探頭為新型的高階陣元。在與以往 Predicate 線陣探頭進行 SE 等同性比對時，我們應如何有技巧地對抗「數位波束合成器硬體信號通道數增加」引起的聲能熱危害疑慮，證明其被大案包絡所遮蔽？
6.3 生物學安全與探頭再處理 (Biocompatibility & Probe Reprocessing)
問題九 [高階消毒材料老化確效]： 生物相容性（ISO 10993）確效常涉及探頭與消毒劑的材料化學相容。當 CMV1-10Z 連續高階 OPA 浸泡 100 次循環後，透鏡（Transducer Lens）是否會發生微開裂、引起微量電流洩漏？我們該如何編排其防電擊（BF型防護）冗餘證明？
問題十 [腔內黏膜短暫接觸豁免論述]： 針對經陰道探頭與黏膜接觸的短暫生物學危害，若原廠黏膜刺激動物實驗缺失，我們能如何引述「FDA 2023 已公佈之醫用特製合成護套阻絕材料（Disposable Probe Sheath Barrier）」來申請豁免原探頭表面材料之刺激重測？
問題十一 [GLP 級實驗室化學比對]： 若 TFDA 堅持不予豁免，我們的生物學確效計畫書（B-Protocol）應如何精確挑選符合大台雙邊共同認可（OECD GLP 相互承認機制）之亞太區實驗室，以規劃最經濟且一次過關的試驗？
6.4 臨床前性能與量測精度物理實驗驗證 (Physics & Measurement Verification)
問題十二 [都卜勒流體仿真幻影誤差]： 對於血管流速計量準確性缺失，我們規劃的物理測試幻影（Phantom）應如何精準對接「校準追溯 NIST 認證之三維精細多普勒泵（Doppler String/Fluid Phantom）」，以提供符合標準差評估的臨床前可信度？
問題十三 [軸向與側向解析力極限宣告]： CMV1-10 在二維高頻切面下的軸向與側向解析度規格（Axial/Lateral Resolution），在沒有原廠物理量測原始底片下，我們的 AI 魔法應如何有技巧地起草「原廠模擬極限數值化學等同證明（Simulation Homology Document）」？
問題十四 [半定量腫瘤硬度輔助診斷標準]： 對於 HERA Z20 特有的 ELASTOSCAN+（半定量彈性成像），該影像之組織抗壓性測量誤差極限應引用何種「國際乳房超音波計量學基準（Breast Ultrasound Biomarker Standard）」來證明其實質等同？
6.5 人工智慧 (AI/ML) 醫療器材 CADe/CADx 深度審查與防退件防線 (AI/ML Regulatory Safety)
問題十五 [ViewAssist 卷積神經網絡白箱申辯]： 為何 ViewAssist 能夠透過 2D 影像進行完美的自動胎兒解剖平面識別？我們若透露過多 U-Net 模型特徵層與卷積層降維參數，是否會產生「原廠核心演算法智慧財產（IP）洩露」？AI 如何在高透明度白箱披露與商業機密間游刃有餘地拿捏其文字分寸？
問題十六 [多中心影像標註一致性 Kappa 值]： 美台官方皆高度注重「影像標註者之間的一致性（Inter-rater Reliability）」。我們在 AI 技術文宣中宣稱的百萬張清洗影像，應如何科學編列標註專家、副教授級和教授級三路盲評的一致性統計模型（如 Cohen's Kappa > 0.85 的統計信心宣告）？
問題十七 [人口學特徵多多樣性代表性]： 審查委員若指控 ViewAssist 與 HeartAssist 的主要開發數據集「缺乏台灣本地孕婦人群的多樣性代表性，有臨床泛化（Generalization）失準風險」，我們應如何引用「亞洲骨骼發達特徵比對等同分析（Asian Homology Anthropometric Study）」來申述泛化能力？
問題十八 [獨立測試數據集絕對隔離證明]： 如何向資訊專家證明，我們的測試數據（Test Set）完全未曾在模型早期交叉驗證或模型挑選中被「污染或數據洩露（Data Leakage）」？我們需要檢附何種數據儲存庫目錄結構和哈希校驗碼（SHA-256 Checklist）？
問題十九 [後上市 Model Drift 動態校準程序]： ViewAssist 面對不同醫院的儀器老化、噪聲，可能引發機器學習性能衰退。我們的「模型漂移監控管計畫」中，應提供何種「端點式主動安全性反饋警報系統（On-device Model Drift Trigger System）」的軟體工程規格？
問題二十 [AI 軟體版本微小迭代不申報辯解]： 當 HERA Z20 原廠發布了 ViewAssist 的 v2.5 更新（相比 FDA 核准 K241971 時的 v2.4 進行了微小權重修正），在台灣查驗登記中我們如何為「此修正不影響預期臨床主要有效性，無須重新提出大案變更申報（Letter to File Scheme）」提供強力、牢靠的法規理據辯護？
