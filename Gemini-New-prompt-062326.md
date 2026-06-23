Hi Hi please create a wow agentic app that user can provide (paste or upload) 510(k) submission materials(txt, md, pdf, json). Then user can paste or upload skill.md or choose the default skill.md. Then agent (default gemini-3.1-flash-lite, user can select models) will use the user provided skill.md to create a interactive review report (web page, html) that user can modify results, adding review notes and export report in google doc, markdown, html. Please adding 3 additional wow ai features to this app. Please also adding wow visualization effects for llm execution, interactive indicators, live log and wow dashboard including 6 wow graphs. Please don;t create code and only create a comprehensive technical specification of the design in markdown in 3000 to 4000 words.  This app will be deploy on hugging face space using streamlit, geminiapi, openai api, agents.yaml, skill.md, please make gemini-3.1-flash-lite a default model for llm features (user can select other gemini, chatgpt models) Ending with 20 comprehensive follow up question.  default skill.md: ---

name: medical-device-review-agent

description: A skill for executing comprehensive technical and administrative reviews on medical device 510(k) registration and submission materials. Use this skill whenever a user provides submission dossiers, device catalogs, manufacturing agreements, QMS/QSD/GMP certificates, or labeling drafts for TFDA/FDA regulatory review. The skill maps materials against the "醫療器材許可證核發與登錄及年度申報準則" and generates a production-ready, interactive web-based review report.

---



# Medical Device 510(k) Registration & Regulatory Review Agent



A highly specialized skill designed to automate, evaluate, and structure technical and administrative compliance reviews for medical device registration dossiers (Traditional 510(k) pathway) in accordance with Taiwan TFDA regulations and the official *醫療器材許可證核發與登錄及年度申報準則*.



## 1. Trigger Contexts & Intent

Activate this skill whenever the user provides text data, files, or explicit review prompts concerning:

* Medical device registration applications (查驗登記申請書).

* Certificates of Free Sale and Manufacture (製售證明 / CFSM).

* Letters of Authorization (原廠授權登記書 / LoA).

* Quality Management System documents (QSD / GMP / ISO 13485).

* Device labeling, IFUs, catalogs, or packaging artwork drafts (標籤、說明書、原廠型錄).

* Pre-clinical technical testing dossiers (IEC 60601-1, IEC 60601-1-2, ISO 10993, IEC 62304, Cybersecurity, Risk Management).



---



## 2. Advanced WOW AI Core Systems



### 

￼

 AI Feature 1: Multi-Agent Discrepancy Reconciliation Engine

* **Mechanism**: Automatically extracts textual strings representing corporate entities, manufacturing facilities, and suites/street designations across disjointed sub-dossiers (Application Form vs. Free Sale Certificate vs. QSD).

* **Action**: Computes semantic and geometric string distances to flag implicit address mismatches (e.g., "Suite 401" vs "4F, Rm 401") and assigns a compliance confidence level before the human reviewer audits the row.



### 

￼

 AI Feature 2: Automated Adaptive-vs-Locked Algorithm Audit Matrix

* **Mechanism**: Targets Machine Learning Medical Device Software (SaMD/SiMD). Scans the software verification architecture, functional specifications, and user manuals to determine whether the algorithm utilizes adaptive updates (continuous post-market learning) or is locked.

* **Action**: If adaptive vectors are detected, the system automatically flags the absence of a "Post-Market Performance Monitoring and Verification Protocol" (上市後性能監測與驗證機制) and drafts a tailored regulatory deficiency notice.



### 

￼

 AI Feature 3: Deep Cross-Dataset Data Contamination Detector

* **Mechanism**: Inspects the validation methodology section of AI/ML software test reports. It extracts metadata schemas belonging to the training datasets and cross-references them mathematically against validation/testing datasets.

* **Action**: If statistical signatures or patient identifiers leak from the training dataset into the test dataset ($Training \cap Test \neq \emptyset$), the system raises a high-severity non-compliance alert indicating data contamination.



---



## 3. Comprehensive Regulatory Checklist (審查指引明細)

*(Checklist remains aligned with the卫福部 TFDA "醫療器材許可證核發與登錄及年度申報準則" tracking 8 primary chapters including Administrative Forms, QMS/QSD cross-matching, Labeling drafts, and specific Technical testing standards like IEC 60601-1, IEC 60601-1-2, and IEC 62304.)*



---



## 4. Output Specification & Structural Code Layout

The agent must output the final report embedded inside a fully self-contained interactive Single Page Application (SPA) HTML template using Tailwind CSS. It must support state retention, custom reviewer note editing, and seamless multi-format exports (Markdown, Google Doc scratchpad text, and Printable raw PDF). sample report (web page):<!DOCTYPE html>

<html lang="zh-TW">

<head>

    <meta charset="UTF-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>510(k) 醫療器材查驗登記互動式審查清單</title>

    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

    <style>

        .status-btn.active-pass { background-color: #10B981; color: white; border-color: #10B981; }

        .status-btn.active-fail { background-color: #EF4444; color: white; border-color: #EF4444; }

        .status-btn.active-na { background-color: #6B7280; color: white; border-color: #6B7280; }

    </style>

</head>

<body class="bg-slate-50 text-slate-800 font-sans min-h-screen flex flex-col">



    <header class="bg-white border-b border-slate-200 sticky top-0 z-50 shadow-sm transition-all">

        <div class="max-w-7xl mx-auto px-4 py-4 sm:px-6 lg:px-8 flex flex-col md:flex-row md:items-center md:justify-between gap-4">

            <div>

                <h1 class="text-xl font-bold text-slate-900 flex items-center gap-2">

                    

￼

 510(k) 醫療器材傳統查驗登記審查工作台

                </h1>

                <p class="text-xs text-slate-500 mt-1">符合 TFDA 最新醫療器材管理法規、QMS 準則與資安指引</p>

            </div>

            <div class="flex flex-wrap items-center gap-4 bg-slate-50 p-2.5 rounded-xl border border-slate-200">

                <div class="text-xs font-semibold px-2 py-1 bg-white rounded shadow-xs">

                    總進度: <span id="progress-percent" class="text-blue-600 font-bold">0%</span>

                </div>

                <div class="flex gap-3 text-xs">

                    <span class="flex items-center gap-1 text-emerald-600">● 通過: <strong id="count-pass">0</strong></span>

                    <span class="flex items-center gap-1 text-rose-600">● 缺失: <strong id="count-fail">0</strong></span>

                    <span class="flex items-center gap-1 text-slate-500">● 免附: <strong id="count-na">0</strong></span>

                    <span class="flex items-center gap-1 text-amber-500">● 待審: <strong id="count-pending">0</strong></span>

                </div>

                <button onclick="exportDeficiencyLetter()" class="bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-medium py-1.5 px-3 rounded-lg transition shadow-xs cursor-pointer flex items-center gap-1">

                    

￼

 彙整缺失通知書

                </button>

            </div>

        </div>

    </header>



    <main class="max-w-7xl mx-auto px-4 py-6 sm:px-6 lg:px-8 flex-grow w-full grid grid-cols-1 lg:grid-cols-3 gap-6">

       

        <div class="lg:col-span-2 space-y-4" id="checklist-container">

            </div>



        <div class="lg:col-span-1">

            <div class="sticky top-28 bg-white border border-slate-200 rounded-xl shadow-xs p-4 space-y-4">

                <h2 class="text-sm font-bold text-slate-900 border-b border-slate-100 pb-2 flex items-center gap-2">

                    

￼

 即時補件缺失彙整區

                </h2>

                <p class="text-xs text-slate-500">當您將項目勾選為「缺失(Fail)」並填寫審查意見後，該缺失條款會自動格式化加入下方文字框，方便複製開立補件公文。</p>

                <textarea id="deficiency-output" class="w-full h-96 p-3 text-xs font-mono bg-slate-900 text-emerald-400 rounded-lg border border-slate-800 focus:outline-none focus:ring-1 focus:ring-indigo-500" readonly placeholder="目前尚未判定任何缺失項。當點擊「缺失」並輸入審查筆記後，此處將自動生成結構化補件意見..."></textarea>

                <div class="flex gap-2">

                    <button onclick="navigator.clipboard.writeText(document.getElementById('deficiency-output').value); alert('缺失文本已複製至剪貼簿！');" class="flex-1 bg-slate-800 hover:bg-slate-950 text-white text-xs font-medium py-2 rounded-lg transition text-center cursor-pointer">

                        🗐 複製文字

                    </button>

                    <button onclick="resetAllProgress()" class="bg-rose-50 hover:bg-rose-100 text-rose-600 text-xs font-medium py-2 px-3 rounded-lg transition cursor-pointer">

                        清空重置

                    </button>

                </div>

            </div>

        </div>

    </main>



    <footer class="bg-white border-t border-slate-200 mt-12 py-4 text-center text-xs text-slate-400">

        510(k) Pre-market Notification Interactive Review System • Technical Framework Design

    </footer>



    <script>

        // Structural Legal & Technical Metadata Array Mapping the TFDA Checklist Dataset

        const checklistData = [

            {

                id: "sec1",

                title: "1、申請書資訊核對",

                items: [

                    { id: "1-1", text: "申請書須以中、英文繕打，並依據送審資料詳實填表。各欄位詳細填寫，並加蓋醫療器材商及負責人印鑑。" },

                    { id: "1-2", text: "載明產品中文及英文名稱、型號、規格，須與製售證明及授權書相符。", note: "審查員得依製售證明及授權書等內容更正申請書誤植處(請於右欄/筆記說明更正依據)。" },

                    { id: "1-3", text: "品名核定原則確認：不得使用他人商標/廠商名稱；不得與他廠相同或近似致混淆；不得有虛偽、誇大、不當聯想；中文品名不得夾雜外文/數字(特例除外)；外銷品名不得與內銷相同。" },

                    { id: "1-4", text: "品名冠有商標者，應檢附商標註冊資料；冠有其他醫療器材商名稱或商標者，應檢附被加冠者同意函。" },

                    { id: "1-5", text: "載明申請醫療器材商名稱、地址，須與醫療器材商許可執照相符。", note: "審查員得自行上網查證地址符合性(例如 Suit A 和 STE A)。" },

                    { id: "1-6", text: "載明製造業者之名稱、地址：國產者須與QMS證明及製造業執照相符；輸入者須與製售、授權書及QMS證明相符。", note: "同集團的 O 和 P 得不需委託合約書。審查員得自行上網查證地址符合性。" },

                    { id: "1-7", text: "屬委託製造者，須分別列出委託者及受託製造業者之名稱、地址 【Manufactured by A for B】。" },

                    { id: "1-8", text: "非屬相同系列名稱 (以原廠說明書之原名為準) 之不同型號(Model或Type)，應另案辦理查驗登記。" }

                ]

            },

            {

                id: "sec2",

                title: "2、陸輸醫療器材與資格證明文件",

                items: [

                    { id: "2-1", text: "列載醫療器材所屬 CCC Code。" },

                    { id: "2-2", text: "如屬國貿局規定之限制輸入產品 (如 MP1、MW0)，應另檢附國貿局准許輸入之證明文件。" },

                    { id: "2-3", text: "輸入業者：應檢附營業項目包含“輸入”之醫療器材販賣業許可執照影本(須於非登不可系統確認登錄)。" },

                    { id: "2-4", text: "製造業者：應檢附醫療器材製造業許可執照影本。" },

                    { id: "2-5", text: "國內委託製造者：應檢附委託者及受託製造業者之醫療器材商許可執照影本。" }

                ]

            },

            {

                id: "sec3",

                title: "3、製售證明（輸入業者專用）",

                items: [

                    { id: "3-1", text: "應為正本 (若為影本，應載明正本所在案號，以供查核)。", note: "請 TFDA 承辦人提供資訊供審查員確認正本是否在署內。" },

                    { id: "3-2", text: "由產製國最高衛生主管機關出具(包含委託製造、未於受託國上市之替代方案、自由販賣證明與製造證明替代組合等特殊法規路徑)。" },

                    { id: "3-3", text: "經我國駐該國外交、商務單位驗證。", note: "與我國訂有技術合作協議國家(美、加、澳)出具者或經中央主管機關認定者，得免驗證。" },

                    { id: "3-4", text: "非英文者，應檢附中文或英文譯本，且須經驗證。" },

                    { id: "3-5", text: "載明該產品製造情形及核准在其本國販賣實況。" },

                    { id: "3-6", text: "自出具日起二年內有效。", note: "逾期也應核對產品型號等資訊，並開立缺失。" },

                    { id: "3-7", text: "載明製造業者名稱、地址，應與申請書一致（審查員得自行上網查證地址符合性）。" },

                    { id: "3-8", text: "載明該醫療器材之名稱、規格型號。", note: "若未列出配件，得僅刊載主體；若有列出配件，應確認含本案申請型號。" }

                ]

            },

            {

                id: "sec4",

                title: "4、國外原廠授權登記書（輸入業者專用）",

                items: [

                    { id: "4-1", text: "應為正本(若為影本，應載明正本所在案號以供查核)。" },

                    { id: "4-2", text: "由製造業者出具，或符合總公司授權代理、原製造廠核發國外代理商再轉授權之合法鏈結證明。" },

                    { id: "4-3", text: "自出具日起一年內有效。" },

                    { id: "4-4", text: "載明製造業者名稱、地址，應與申請書一致（審查員得上網核對 Suite/STE 等地址縮寫）。" },

                    { id: "4-5", text: "載明原製造業者授權我國代理商申請查驗登記，並配合我國代理商符合醫療器材相關管理規定。" },

                    { id: "4-6", text: "如非以英文出具者，應檢附中文或英文譯本。" },

                    { id: "4-7", text: "指明委託或授權登記醫療器材之名稱、規格型號等（或註明授權原廠所有產品）。" },

                    { id: "4-8", text: "載明被授權登記醫療器材商名稱暨地址（須與國貿局基本資料及醫材商許可執照相符）。" },

                    { id: "4-9", text: "如屬委託製造者，原廠授權登記書應由委託者出具，且應載明委託者及受託製造業者之名稱、地址。" }

                ]

            },

            {

                id: "sec5",

                title: "5、品質管理系統證明文件（QSD / GMP）",

                items: [

                    { id: "5-1", text: "符合「醫療器材品質管理系統準則」之證明文件影本(新改列醫材者緩衝期內得以GMP替代)。" },

                    { id: "5-2", text: "持有醫療器材商應與查驗登記申請者一致；若不同，須檢附認可登錄證明函、持有商授權證明及原廠授權書。" },

                    { id: "5-3", text: "所載製造業者名稱、地址須與申請書一致。" },

                    { id: "5-4", text: "自登錄日起，有效期間為 3 年。" },

                    { id: "5-5", text: "所載品項須與申請產品屬同一分類品項。", note: "作業得不含設計，但需確認製造與最終驗放。SaMD得不含製造。若僅有「設計、包裝、貼標及最終驗放」應附流程圖與關係說明、必要時要求加備QSD。" },

                    { id: "5-6", text: "如擬申請品項與原登錄品項不儘相符，須附原廠說明函說明屬相同類別且品質系統一致。" },

                    { id: "5-7", text: "國內製造業者若將其中製造、滅菌程序委託他廠，應另檢附受託廠符合醫療器材QMS之證明文件。" },

                    { id: "5-8", text: "涉及包裝、貼標或最終驗放委託，且於許可證刊載委託製造者，應檢附受託廠符合QMS之證明。" }

                ]

            },

            {

                id: "sec6",

                title: "6、委託製造相關合約與證明文件",

                items: [

                    { id: "6-1", text: "涉及委託製造者，應檢附委託製程說明文件(含各製程受託廠之名稱、地址並核對地址符合性)。" },

                    { id: "6-2", text: "國內委託全部製程、製造或滅菌者，應檢附經 TFDA 核准之委託製造證明文件，載明之名稱地址與品項須相符。" },

                    { id: "6-3", text: "國內委託非屬作業準則項目但涉及許可證刊載者，應檢附雙方有效期限內之委託製造契約文件。" },

                    { id: "6-4", text: "國外業者委託製造，應出具雙方有效之委託製造契約。若製售證明已可佐證委受託關係，得免附雙方契約。" }

                ]

            },

            {

                id: "sec7",

                title: "7、標籤及說明書（審查核心）",

                items: [

                    { id: "7-1", text: "產品中文標籤、說明書或包裝擬稿 2 份。須標示批號/序號、製造日期、效期、醫材商及製造廠名稱地址。", note: "醫材商地址應加註不核定標籤警語備註。滅菌產品應提供中文滅菌標籤。" },

                    { id: "7-2", text: "原廠標籤、包裝 2 份(應刊載原產國別)。原則上應附各型號資料，具一致性之眾多配件得提代表型號。標籤應實貼，不得重疊。" },

                    { id: "7-3", text: "原廠型錄及產品英文/原文使用說明書。" },

                    { id: "7-4", text: "產品實際外觀彩色圖片 2 份(選配配件需提供彩色圖片，特殊狀況得由代表型號佐證)。" },

                    { id: "7-5", text: "中文型錄、說明書擬稿 2 份。須依據原廠資料詳實翻譯效能、適應症、警告、型號規格，不得誇大或擴大範圍。", note: "開立缺失時須明確說明修改內容與其所在之頁碼章節。補件時須交回並彩色列印。SaMD 應列出軟體版本。" },

                    { id: "7-6", text: "全部製程委託製造者，標籤、說明書或包裝應依規定刊載受託者及委託者名稱地址，且與申請書完全一致。" }

                ]

            },

            {

                id: "sec8",

                title: "8、技術審查資料（安全與功能性驗證）",

                items: [

                    { id: "8-1", text: "電性安全與電磁相容性(IEC 60601-1 / IEC 60601-1-2)：不接受僅檢附證書，須檢附完整測試報告。", note: "若實驗室 ISO/IEC 17025 認證有疑義，應至 TAF 或國際機構查詢範圍並列印。" },

                    { id: "8-2", text: "生物相容性試驗：依接觸部位與時間提供評估報告。具輸注藥物/化療功能者需加附藥物相容性與溶出/穩定性測試。" },

                    { id: "8-3", text: "功能性試驗：依臨床前測試基準、美國FDA指引、510(k) summary 等檢附。重要項目引用國際標準者應檢附完整報告。" },

                    { id: "8-4", text: "特定設備安全規範確認：警報(IEC 60601-1-8)、雷射(IEC 60825-1)、光生物安全(IEC 62471)、眼科儀器(ISO 15004-1/-2)、超音波安全(IEC 60601-2-37, 熱/機械指數最大值)、X光設備(輻射防護、球管、平板偵測器)、輸液幫浦(IEC 60601-2-24)。" },

                    { id: "8-5", text: "滅菌與安定性確效：EO滅菌配件須檢附三年內確效報告與批次放行紀錄；耗材與隱形眼鏡應檢附安定性試驗報告。" },

                    { id: "8-6", text: "軟體確效與網路安全(SaMD)：檢附完整報告而非摘要。版次不同應確認是否屬大變更。涉及外部雲端需附 ISO 27001 與加密測試。說明書應符合我國《個人資料保護法》警語與網安配置建議。" },

                    { id: "8-7", text: "AI / 演算法醫材審查：說明屬演進式(Adaptive)或閉鎖式(Locked)；訓練資料集與測試資料集(Test dataset)必須完全獨立隔離；獨立性能評估資料庫須確認是否為原始或預處理資料。" },

                    { id: "8-8", text: "清潔確效與重處理(Reprocessing)：可重複使用耗材/超音波探頭等，須檢附清潔、消毒/滅菌及重處理後功能性測試。" }

                ]

            }

        ];



        // System State

        let reviewStates = JSON.parse(localStorage.getItem('510k_review_states')) || {};



        // Initialization

        window.addEventListener('DOMContentLoaded', () => {

            renderChecklist();

            calculateProgress();

        });



        // Render Dom Elements Dynamically

        function renderChecklist() {

            const container = document.getElementById('checklist-container');

            container.innerHTML = '';



            checklistData.forEach(section => {

                const sectionEl = document.createElement('div');

                sectionEl.className = "bg-white border border-slate-200 rounded-xl shadow-xs overflow-hidden";

               

                // Header of Accordion Section

                let headerHtml = `

                    <div class="bg-slate-100 px-4 py-3 border-b border-slate-200 flex justify-between items-center cursor-pointer" onclick="toggleSection('${section.id}')">

                        <h3 class="text-sm font-bold text-slate-800">${section.title}</h3>

                        <span id="icon-${section.id}" class="text-xs text-slate-400 font-mono">▼</span>

                    </div>

                    <div id="body-${section.id}" class="divide-y divide-slate-100 transition-all">

                `;



                // Content Rows

                section.items.forEach(item => {

                    const state = reviewStates[item.id] || { status: 'pending', note: '' };

                   

                    headerHtml += `

                        <div class="p-4 hover:bg-slate-50/50 transition flex flex-col md:flex-row gap-4 items-start">

                            <div class="flex-grow space-y-1">

                                <div class="text-xs font-semibold text-slate-400 font-mono">[條款 ${item.id}]</div>

                                <p class="text-xs font-medium text-slate-700 leading-relaxed">${item.text}</p>

                                ${item.note ? `<p class="text-[11px] text-amber-600 bg-amber-50 border-l-2 border-amber-400 px-2 py-1 mt-1 rounded-xs font-mono">

￼

 審查提示：${item.note}</p>` : ''}

                            </div>

                           

                            <div class="w-full md:w-72 flex flex-col gap-2 shrink-0">

                                <div class="grid grid-cols-3 gap-1 bg-slate-100 p-0.5 rounded-lg border border-slate-200">

                                    <button onclick="setStatus('${item.id}', 'pass')" class="status-btn cursor-pointer text-center py-1 text-[11px] font-bold rounded-md border border-transparent transition ${state.status === 'pass' ? 'active-pass' : 'text-slate-600 hover:bg-white'}">Pass</button>

                                    <button onclick="setStatus('${item.id}', 'fail')" class="status-btn cursor-pointer text-center py-1 text-[11px] font-bold rounded-md border border-transparent transition ${state.status === 'fail' ? 'active-fail' : 'text-slate-600 hover:bg-white'}">Fail</button>

                                    <button onclick="setStatus('${item.id}', 'na')" class="status-btn cursor-pointer text-center py-1 text-[11px] font-bold rounded-md border border-transparent transition ${state.status === 'na' ? 'active-na' : 'text-slate-600 hover:bg-white'}">N/A</button>

                                </div>

                                <textarea

                                    id="note-${item.id}"

                                    oninput="updateNote('${item.id}', this.value)"

                                    class="w-full h-16 p-1.5 text-xs bg-slate-50 border border-slate-200 rounded-lg focus:outline-none focus:ring-1 focus:ring-indigo-500 placeholder-slate-400"

                                    placeholder="輸入審查意見或具體補件缺失理由..."

                                >${state.note}</textarea>

                            </div>

                        </div>

                    `;

                });



                headerHtml += `</div>`;

                sectionEl.innerHTML = headerHtml;

                container.appendChild(sectionEl);

            });

        }



        // Accordion Collapse Action

        function toggleSection(id) {

            const body = document.getElementById(`body-${id}`);

            const icon = document.getElementById(`icon-${id}`);

            if(body.classList.contains('hidden')) {

                body.classList.remove('hidden');

                icon.innerText = "▼";

            } else {

                body.classList.add('hidden');

                icon.innerText = "►";

            }

        }



        // Change Status Control

        function setStatus(itemId, status) {

            if (!reviewStates[itemId]) reviewStates[itemId] = { status: 'pending', note: '' };

            reviewStates[itemId].status = status;

           

            // Auto fill standard text if failed and empty to accelerate reviewer velocity

            if (status === 'fail' && !reviewStates[itemId].note) {

                reviewStates[itemId].note = "經審查，此項資料未臻完備，請依據法規指引補正相關佐證資料。";

                const textarea = document.getElementById(`note-${itemId}`);

                if(textarea) textarea.value = reviewStates[itemId].note;

            }



            saveState();

            renderChecklist();

            calculateProgress();

        }



        // Sync Notes Text

        function updateNote(itemId, value) {

            if (!reviewStates[itemId]) reviewStates[itemId] = { status: 'pending', note: '' };

            reviewStates[itemId].note = value;

            saveState();

            exportDeficiencyLetter();

        }



        // Save State into Local Storage

        function saveState() {

            localStorage.setItem('510k_review_states', JSON.stringify(reviewStates));

        }



        // Calculate and Render Metrics Progress Tracker Realtime

        function calculateProgress() {

            let totalItems = 0;

            checklistData.forEach(s => totalItems += s.items.length);



            let pass = 0, fail = 0, na = 0, pending = 0;



            checklistData.forEach(s => {

                s.items.forEach(item => {

                    const state = reviewStates[item.id]?.status || 'pending';

                    if(state === 'pass') pass++;

                    else if(state === 'fail') fail++;

                    else if(state === 'na') na++;

                    else pending++;

                });

            });



            document.getElementById('count-pass').innerText = pass;

            document.getElementById('count-fail').innerText = fail;

            document.getElementById('count-na').innerText = na;

            document.getElementById('count-pending').innerText = pending;



            const reviewedCount = pass + fail + na;

            const percent = totalItems > 0 ? Math.round((reviewedCount / totalItems) * 100) : 0;

            document.getElementById('progress-percent').innerText = `${percent}%`;



            exportDeficiencyLetter();

        }



        // Compile and Auto Build the Dynamic Deficiency Markdown Notification Letter

        function exportDeficiencyLetter() {

            let letter = `【510(k) 查驗登記不准予/補件通知書 - 缺失條款彙整模組】\n`;

            letter += `發文日期：中華民國 115 年 (2026年) \n`;

            letter += `審查結果摘要：本案送審文件經實質審查後，計有下列條款未達法定技術合規標準，請依說明事項逐項補正：\n`;

            letter += `==================================================\n\n`;



            let hasFailures = false;



            checklistData.forEach(section => {

                let sectionAdded = false;

                section.items.forEach(item => {

                    const state = reviewStates[item.id];

                    if(state && state.status === 'fail') {

                        hasFailures = true;

                        if (!sectionAdded) {

                            letter += `■ [${section.title}]\n`;

                            sectionAdded = true;

                        }

                        letter += `  

￼

 缺失條款 ${item.id} 規範：${item.text}\n`;

                        letter += `  

￼

 審查意見 / 補件要求：${state.note || '請補充說明細節。'}\n`;

                        letter += `  ------------------------------------------------\n`;

                    }

                });

            });



            if(!hasFailures) {

                letter += `（目前無缺失。當您點選任一項目為「Fail」並填寫筆記後，此處將同步即時自動產出缺失報告文本。）`;

            }



            document.getElementById('deficiency-output').value = letter;

        }



        // Reset System Data Hook

        function resetAllProgress() {

            if(confirm("確定要清空所有目前的審查進度與筆記記錄嗎？")) {

                reviewStates = {};

                saveState();

                renderChecklist();

                calculateProgress();

            }

        }

    </script>

</body>

</html>
