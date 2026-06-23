RegulatoryAI 510(k) 智慧醫療器材法規審查工作台：使用者操作與本地部署指南
歡迎使用 RegulatoryAI 510(k) Agentic Workspace (v2.0)。本系統是專為醫療器材法規事務（RA）專家與審查員設計的企業級 AI 代理（Agentic）工作台。系統預設採用高效能、低延遲的 gemini-3.1-flash-lite 模型，並支援切換至高階 Gemini Pro 及 OpenAI GPT 系列模型。

本指南將分為兩大部分：第一部分：系統功能操作指南（協助您掌握核心審查與 Meta-Skill 優化模組）；第二部分：新手本地端部署教學（從零開始在您自己的電腦上運行此系統）。

第一部分：系統功能操作指南
本工作台由兩個核心主模組（Module）構成，您可在 Streamlit 側邊欄或頂部標籤頁進行切換：

[模組 A] Core 510(k) 查驗登記審查工作台：負責實際審查送審文件、呈現互動式網頁報告與動態儀表板。

[模組 B] Meta-Skill 代理人技能優化沙盒：負責動態調整、升級與測試審查邏輯基準（skill.md）。

1. 模組 A：核心 510(k) 審查流程
步驟 1：上傳送審物料
進入「Core 510(k) Workspace」主界面。

在文件上傳區，您可以批次拖曳或上傳醫材廠的送審資料。系統支援 .pdf（原始報告/證書）、.md（說明書）、.txt（規格書） 與 .json（結構化資料）。

您可以選擇維持預設的 gemini-3.1-flash-lite 模型以獲得極速審查體驗，或在側邊欄切換為 gpt-4o 以處理極度複雜的臨床數據。

步驟 2：觀看 AI 即時執行日誌與執行狀態
點擊 「開始法規實質審查」。

視覺化狀態燈 將開始閃爍（例如：[PARSING] 讀取中、[CROSS-REFERENCING] 交叉比對中）。

頁面下方的 「即時邏輯日誌（Live Logs）」 會以串流（Streaming）方式跑出底層 AI 代理人的思考軌跡（例如：正在計算製造廠地址的 Jaro-Winkler 文本距離、或正在抓取演算法是否為持續學習式之特徵）。

步驟 3：互動式審查報告（SPA HTML）操作
審查完成後，中央區域會渲染出符合衛福部 TFDA《醫療器材許可證核發與登錄及年度申報準則》的八大章節互動清單。

狀態判定（Pass / Fail / N/A）：AI 會預先為各條款勾選判定。若您認為某項缺失需要調整，可手動點擊切換。當您將某項切換為 Fail 時，系統會自動在右側筆記欄填入標準法規補件提示語。

即時補件缺失彙整區（右側面板）：當您在左側清單修改任何審查意見時，右側的黑色文字框會即時（Real-time）自動同步生成結構化的「不准予/補件通知書」Markdown 文本，完全無需手動複製剪貼。

步驟 4：觀看 6 大 WOW 視覺化儀表板
切換至儀表板分頁，查看本次送審文件的深度統計：

語意差異紅外線矩陣：一眼看出申請書、製售證明（CFSM）與品質管理系統（QSD）之間，工廠地址與公司名稱的字面漂移與嚴重度（紅色越深代表字面差異越大）。

條款合規雷達圖：展示八大章節的合規分數分佈，立刻暴露哪一章節（如：標籤說明書或技術資安）缺失最多。

Token 消耗與延遲波形圖：監控 AI 運算成本與回應速度。

數據污染重疊分佈圖：若為 AI/ML 醫材，此圖會揭示訓練資料集與測試資料集是否存在 patient ID 洩漏或交叉污染。

演算法變異風險象限圖：評估軟體（SaMD）的演算法更新頻率與上市後效能監測機制的風險匹配度。

系統關鍵缺失堆疊圖：依據嚴重等級（Critical / Minor / Missing）加總呈現當前案件的整體風險指標。

步驟 5：匯出審查報告
在側邊欄或報告底部，點擊對應按鈕匯出結果：

匯出 Markdown (.md)：存為純文字檔，供內部系統封存。

傳送至 Google Doc 暫存區：透過 API 直接線上建立一份乾淨的補件公文草稿。

下載獨立 SPA 網頁 (.html)：下載一個自包含（Self-contained）的精美網頁檔案。此檔案下載後脫離伺服器仍具備互動功能，可任意點擊勾選並自帶 LocalStorage 記憶功能。

2. 模組 B：Meta-Skill 代理人技能優化沙盒
當法規更新、或是您希望引導 AI 審查特定新型醫材（例如：2026年最新的動態隱私資安指引）時，您需要調整 skill.md。

步驟 1：上傳與優化 Skill
切換至「Meta-Skill Optimization Engine」分頁。

在左側編輯器上傳或直接貼上現有的 skill.md。

在優化指令輸入框輸入您的期望（例如：「請在第 8 章技術審查中，加入針對雲端 SaMD 的 ISO 27001 與個人資料保護法警告語審查邏輯」），然後點擊 「優化 Skill 邏輯」。

步驟 2：比對版本差異（Visual Diff）
AI 執行完畢後，界面會跑出側邊雙面板對照（Side-by-Side Diff）。

綠色區塊代表 AI 幫您新增的法規查核點，紅色代表刪除的過時條款。

您可以直接在右側的最終預覽框內進行人工微調與文字修正。

步驟 3：隔離沙盒測試（Sandbox Test）
為了確保新寫好的 skill.md 查驗能力正常，請勿直接上線。

在下方的「沙盒測試區」，上傳一份簡短的範例文件（例如：某產品的資安宣告書片段）。

點擊 「執行沙盒測試運算」。

系統會啟動一個完全隔離的背景 AI 執行緒，用這份新寫的 skill.md 去審查該範例文件，並將發現的缺失當場列印在沙盒日誌中。

步驟 4：一鍵熱部署與下載
若沙盒測試結果符合預期，點擊 「一鍵熱部署至核心工作台」。系統記憶體會立刻重載，接下來 Module A 的審查就會全部採用這套最新優化的法規邏輯。

點擊 「下載優化版 skill.md」 可將檔案存回本地端留存。

第二部分：新手本地端部署教學（零基礎自學步驟）
如果您想在自己的個人電腦（Windows 或 Mac）上運行這個強大的應用程式，請按照以下步驟逐步操作。我們將從最基礎的環境安裝開始介紹。

部署流程圖解
系統必備環境需求
作業系統：Windows 10/11 或 macOS

網路環境：需連網（以呼叫 Gemini API 或 OpenAI API）

必備憑證：

Gemini API Key（請至 Google AI Studio 免費或付費申請）

OpenAI API Key（選配，若要使用 ChatGPT 模型則需準備）

步驟 1：下載並安裝 Python（系統運行基礎）
Python 是運行此工作台所需的程式語言環境。

對於 Windows 使用者：
前往 Python 官方網站下載頁面。

下載最新的 Python 3.10 或 3.11 版本（建議選用穩定版，暫不建議最新 3.12+ 以免部分科學套件不相容）。

【極重要】 執行安裝檔時，請務必勾選最下方的 "Add python.exe to PATH"（將 Python 加入系統路徑），然後點擊 "Install Now"。

對於 Mac 使用者：
建議使用 Mac 的套件管理器 Homebrew。打開「終端機」（Terminal），輸入以下指令安裝 Python：

Bash
brew install python@3.11
或者直接去 Python 官網下載 .pkg 安裝檔並連點下一步完成安裝。

步驟 2：獲取程式碼與專案結構
在您的電腦中（例如 D 磁碟機 或 文件 資料夾），建立一個名為 RegulatoryAI_App 的新資料夾。

將此專案的所有核心檔案放入該資料夾中。請確認資料夾內的檔案目錄結構如下所示：

Plaintext
RegulatoryAI_App/
├── .streamlit/
│   └── config.toml
├── app.py
├── core_orchestrator.py
├── agents.yaml
├── skill.md
├── requirements.txt
└── components/
    ├── charts.py
    ├── parser.py
    └── skill_optimizer.py
步驟 3：開啟終端機並切換至專案資料夾
我們要讓電腦的命令列界面進入該資料夾。

對於 Windows 使用者：
按下鍵盤上的 Win + R 鍵，輸入 cmd，按 Enter 開啟「命令提示字元」。

輸入您的專案所在路徑（假設放在 D 槽的資料夾）：

DOS
d:
cd RegulatoryAI_App
對於 Mac 使用者：
開啟「終端機」（Terminal）。

輸入 cd 後空一格，然後直接把 RegulatoryAI_App 資料夾用滑鼠拖曳進終端機視窗內，路徑會自動填入，按下 Enter 即可。

步驟 4：建立虛擬環境與安裝相依套件（防套件衝突）
虛擬環境就像一個乾淨的沙盒，能確保此專案的套件不會與您電腦裡的其他程式衝突。

建立虛擬環境（在終端機中輸入以下指令）：

Bash
python -m venv venv
此時資料夾內會多出一個名為 venv 的小資料夾。

啟用虛擬環境：

Windows 系統 請輸入：

DOS
venv\Scripts\activate
Mac / Linux 系統 請輸入：

Bash
source venv/bin/activate
啟用成功後，您的命令列最前面會出現 (venv) 的字樣。

升級基礎工具並安裝所有 AI 套件（請確保電腦此時正連接網路）：

Bash
pip install --upgrade pip
pip install -r requirements.txt
系統會自動下載並安裝 Streamlit、Google GenAI SDK、OpenAI SDK、Plotly 圖表庫、Tailwind 支援等所有必備組件。請靜候 1~3 分鐘直至安裝完成。

步驟 5：配置您的 API 金鑰（環境變數設定）
為保護您的金鑰安全，系統預設從系統環境變數中讀取憑證。

對於 Windows 使用者（設定臨時變數）：
在每次啟動程式前的同一個命令提示字元視窗內，輸入：

DOS
set GEMINI_API_KEY=您的實際Gemini金鑰文字
set OPENAI_API_KEY=您的實際OpenAI金鑰文字（若無可不填）
對於 Mac 使用者（設定臨時變數）：
在終端機視窗內，輸入：

Bash
export GEMINI_API_KEY="您的實際Gemini金鑰文字"
export OPENAI_API_KEY="您的實際OpenAI金鑰文字"（若無可不填）
步驟 6：啟動 RegulatoryAI 工作台應用程式
一切就緒！請在虛擬環境啟用的狀態下，輸入以下終端機指令：

Bash
streamlit run app.py
啟動成功表現：
終端機會顯示：

Plaintext
You can now view your Streamlit app in your browser.
Local URL: http://localhost:8501
您的預設瀏覽器（如 Chrome、Edge 或 Safari）會自動彈出一個新分頁，網址為 http://localhost:8501。

您將看到精美的「510(k) 醫療器材查驗登記智慧審查工作台」繁體中文界面，此時即可開始享受完全運行在您本地端的多代理人智慧法規服務！

第三部分：20 個技術與操作深度延伸思考問題
為了協助系統管理員與法規架構師進一步優化本系統，以下列出 20 個關鍵的技術與操作延伸問題：

新手使用者如果在安裝 requirements.txt 時遇到 C 語言編譯器缺失（例如 pydantic 或科學運算套件錯誤），本地端最快的排解手段為何？

在 Windows 的命令提示字元中設定的 set GEMINI_API_KEY 關閉視窗後就會失效，如何指導一般使用者在作業系統中設定「永久環境變數」？

當 Module B 的 Meta-Agent 在優化 skill.md 時，若使用者輸入的導引提示詞過於模糊（如：「請讓它變得更厲害」），系統內建的提示詞工程如何確保優化品質？

當 Module B 的沙盒測試（Sandbox Test）發現邏輯崩潰或輸出格式不符時，系統有沒有提供一鍵「回滾至前一版安全 Skill」的防錯機制？

本地端執行 Streamlit 時，若使用者上傳了高達 200MB 的多個醫材 PDF 檔案，記憶體（RAM）快取機制該如何動態配置以防瀏覽器當機？

當使用者點擊「熱部署至核心工作台」時，系統底層是如何在不重啟整個 Streamlit 伺服器的情況下，動態更新已載入的 Agent 物件實例？

預設的 gemini-3.1-flash-lite 模型在處理中文法規專有名詞與台灣 TFDA 語境時，是否會出現簡繁體混雜？如何透過 skill.md 的 System Prompt 進行強制校正？

互動式 SPA HTML 報告中的 LocalStorage 狀態儲存機制，若在使用者下載檔案並更換不同電腦或瀏覽器開啟時，該如何實現審查筆記的跨裝置遷移？

本地部署模式下，如果多位公司內部同仁透過區域網路（LAN）連線至同一台機器的 http://IP:8501，Streamlit 的 Session State 會不會互相干擾？

對於內含大量掃描圖片、手寫簽章的 QSD 證明文件 PDF，本地端 parser 該如何流暢地調用開源 OCR（如 Tesseract）或雲端視覺 API 進行光學字元辨識？

6 大視覺化儀表板中的「數據污染重疊分佈圖」，其底層演算法在面對廠商只提供聚合統計圖（Summary Charts）而非原始患者矩陣時，該如何計算重疊指標？

如何防止惡意廠商在送審的 .txt 或 .md 說明書中埋入對抗性提示詞攻擊（Prompt Injection），藉此誘騙審查 Agent 跳過關鍵的安全試驗核對？

當匯出至 Google Docs 時，若使用者電腦沒有配置全域的 Google Workspace OAuth 憑證，系統是否提供了替代的「複製 Raw HTML / Markdown 剪貼簿」方案？

在 Module B 進行 Side-by-Side Diff 視覺比對時，若新舊 skill.md 行數差異極大，Streamlit 的渲染組件如何保持秒級渲染而不卡頓？

針對 IEC 60601-1 的電性安全測試報告，AI 專家的提取規則是否支援自動識別並警示「報告中所引用之測試實驗室已超出 TAF 認證有效期限」？

本系統的 agents.yaml 宣告式架構，未來若想新增一個專門審查「體外診斷醫療器材（IVD）」的獨立 Agent，法規架構師需要修改哪些底層介面？

當使用者選擇 OpenAI 的 o1 強推理模型來審查軟體演算法時，其長思考時間（Reasoning Tokens）所引發的 Streamlit 逾時（Timeout），應如何透過是非同步非阻塞線程（Async Thread）解決？

下載後的 Standalone SPA HTML 報告，若使用者在無網路環境（完全離線）下打開，Tailwind CSS 的 CDN 載入失敗會如何影響排版？有無內建備份樣式？

語意差異紅外線矩陣（Graph 1）所使用的文本距離演算法（如 Jaro-Winkler 或 Levenshtein），在處理中文地址縮寫（如「3樓之1」與「3F-1」）時的權重該如何動態微調？

本地運行此工作台時，所有的 API 請求皆直接發送至 Google 或 OpenAI 的伺服器，對於極度要求資料不外洩的頂級醫療器材廠，系統該如何調度本地開源大模型（如 Llama 3 或 Mistral 透過 Ollama）來完全斷網運行？
