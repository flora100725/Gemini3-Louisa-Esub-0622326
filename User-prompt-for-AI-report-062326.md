Hi please create a agentic app that this app ask user 30 questions about content of a comprehensive 510(k) review report as sample report in markdown in 3000 to 4000 words in Traditional Chinese. Then agent will create a comprehensive 510(k) review report based on users answers as sample report in markdown in 3000 to 4000 words in Traditional Chinese with new content in coral color. Then user can modify the report in markdown view or output to google doc, html, pdf, md. Then user can keep prompt on the report (default gemini-3.1-flash-lite, user can select models) or use ai magics (5 wow ai features created by you). Please also create a new module to let user upload, modify, download questions in markdown. Please adding 3 additional wow ai features to this app. Please don't create code and only create a comprehensive technical specification of this design in markdown in 3000 to 4000 words. Ending with 20 comprehensive follow up questions sample report:sample report:510(k) Technical Review report
Date:

本申請案為新申請案，該產品為診斷式超音波Samsung Medison 超音波診斷系統(型號：HERA Z20)，本案產品已取得美國FDA上市許可(K241971)。美國FDA 510(k)申請案(K241971)包含Samsung Medison Ultrasound systems多個產品型號(HERA Z20, R20, HERA Z30, R30)，本申請案僅包含其中單一型號規格(型號：HERA Z20)。

FDA 510(k) intended use (K241971)
The diagnostic ultrasound system and transducers are designed to obtain ultrasound images and analyze body fluids. 
The clinical applications include: Fetal/Obstetrics, Abdominal, Gynecology, Intra-operative, Pediatric, Small Organ, Neonatal Cephalic, Adult Cephalic, Trans-rectal, Trans-vaginal, Muscular-Skeletal (Conventional, Superficial), Urology, Cardiac Adult, Cardiac Pediatric, Thoracic, Trans-esophageal (Cardiac) and Peripheral vessel. 

審查參考美國FDA指引文件及TFDA臨床前測試基準
[1]Marketing Clearance of Diagnostic Ultrasound Systems and Transducers
Guidance for Industry and Food and Drug Administration Staff, February 2023
https://www.fda.gov/regulatory-information/search-fda-guidance-documents/marketing-clearance-diagnostic-ultrasound-systems-and-transducers
[2] 「診斷用超音波影像系統暨超音波轉換器(探頭)」臨床前測試基準(2018)
[3]醫療器材軟體確效指引
[4] 適用於製造業者之醫療器材網路安全指引
(醫療器材網路安全評估分析參考範本2022)
[5]人工智慧/機器學習技術之電腦輔助偵測(CADe)及電腦輔助診斷(CADx)醫療器材查驗登記技術指引

產品適用標準：
	電性：IEC 60601-1:2025 醫療電氣設備：基本安全與基本性能要求。
	電磁：IEC 60601-1-2:2024 電磁相容測試。
	安規：IEC 60601-2-37	超音波診斷與監控設備的特定安全要求。
	生物相容ISO 10993-1	醫療器材生物學評估：風險管理程序。
	超音波探頭安全：IEC 62359聲場特徵：熱指數與機械指數的確定方法或NEMA UD 2-2004 聲學輸出測量標準
	軟體驗證文件/IEC 62304依據《醫療器材軟體確效指引》。
	網路安全評估依據《適用於製造業者之醫療器材網路安全指引》
	人工智慧/機器學習技術之電腦輔助偵測(CADe)及電腦輔助診斷(CADx)醫療器材查驗登記技術指引

審查重點：
[1]產品電性、電磁測試評估。
	電性：IEC 60601-1:2025 醫療電氣設備：基本安全與基本性能要求。
	電磁：IEC 60601-1-2:2024 電磁相容測試。
[2]超音波產品標準
安規：IEC 60601-2-37	超音波診斷與監控設備的特定安全要求。
	超音波探頭安全：IEC 62359聲場特徵：熱指數與機械指數的確定方法或NEMA UD 2-2004 聲學輸出測量標準
[3]生物相容性評估(所有探頭)
	生物相容ISO 10993-1	醫療器材生物學評估：風險管理程序。
[4]軟體驗證報告
[5]網路安全評估報告
[6]特殊軟體功能測試評估及AI CADe/CADx軟體文件
	人工智慧/機器學習技術之電腦輔助偵測(CADe)及電腦輔助診斷(CADx)醫療器材查驗登記技術指引
	2-1、產品描述 | 預期用途、適用族群、禁忌症、輸出邏輯
	2-2、演算法架構 | 模型類型 (CNN/RNN)、框架、參數設定
	2-3、開發與驗證數據集 | 醫院、設備廠牌、數據清洗流程、標註人員資歷
	數據集規模與來源： 訓練集與驗證集包含多少樣本？數據是從哪些醫療機構或資料庫獲取的？
	人口統計學特徵： 訓練與驗證數據中是否包含足夠多樣的人種、年齡、性別與地理分布？這些特徵是否與器材預期使用的目標人群相符？
	數據獨立性： 贊助商必須描述如何確保測試數據（Test Data）與訓練數據（Training Data）之間的獨立性，以證明模型的泛化能力（Generalization），防止「過擬合」（Overfitting）現象。
	2-4、獨立性能評估(Non-Clinical Performance Testing) | 靈敏度、特異度、AUC、泛化能力證明
	統計信心與不確定性指標：AI 的輸出並非絕對，往往帶有機率性。因此，摘要中應包含對預測結果的「統計信心水準」（Statistical Confidence Level）描述。這可能包括信心區間（Confidence Intervals）、預測值的不確定性度量（Uncertainty Metrics）以及其他能反映模型可靠性的統計工具。
	對比分析： 比較訓練數據集與驗證數據集的差異，並說明這些輸入數據如何代表實際臨床環境中的預期狀況。
	2-5、模型更新與維護機制
	AI 模型具有「持續學習」或「版本迭代」的潛力。贊助商需描述模型在上市後將如何進行維護與更新。這涉及到效能監控、模型漂移（Model Drift）的檢測以及後續更新的驗證程序。

[7]最終產品品質檢測/功能性試驗(Performance test) 
1.各項超音波轉換器(探頭)(Transducer)規格：如中心頻率(或操作頻帶)、軸向解析度、側向解析度等。 
2.各測量類型(如距離、體積、心跳、流速等)量測準確度。
2.熱指數(Thermal index, TI)、機械指數(Mechanical Index, MI)之顯示、準確度須符合IEC 60601-2-37之規範。 
3.對於每一種操作模式(Operation mode)提供熱指數(Thermal index, TI)、機械指數(Mechanical Index, MI)之最大值。

本案產品主要規格(根據中文說明書)：
產品敘述/適應症 
本產品適用於人體診斷超音波成像和體液影像分析。 臨床應用包括：胎兒/產科、腹部、婦科、手術中、小兒科、小器官、新生兒頭部、成人頭部、經直腸、經陰道、肌肉骨骼(常規、淺表組織)、泌尿科、成人心臟、小兒心臟、胸部、經食道(心臟)、周邊血管。


本案產品主要性能特徵 
•	基本技術：數位波束合成、脈衝都卜勒
•	操作模式： 2D, M, 彩色Doppler、能量Doppler/S-Flow 、脈衝Doppler, 連續Doppler, 立體3D/4D, ELASTOSCAN+模式、TDI模式、TDW 模式、MV-FLOW 模式、IOTA-ADNEX 模式、S-DETECT FOR 乳房模式、S-DETECT FOR 甲狀腺模式、AI特殊影像處理(中文說明書記載：系統標準功能、系統選配功能、影像處理)
•	AI 功能： ViewAssist, Live ViewAssist, HeartAssist, EzVolume 
•	探頭：包含新款 CMV1-10, CMV1-10Z

審查紀錄
[1]產品電性、電磁測試評估。
	電性：IEC 60601-1:2025 醫療電氣設備：基本安全與基本性能要求。
OK，檢附Nemko Korea Co., Ltd.出具IEC 60601-1:2025測試評估報告
電磁：IEC 60601-1-2:2014 電磁相容測試。
OK，檢附Nemko Korea Co., Ltd.出具IEC 60601-1-2:2014測試評估報告

[2]超音波產品標準
	安規：IEC 60601-2-37	超音波診斷與監控設備的特定安全要求。
	超音波探頭安全：IEC 62359聲場特徵：熱指數與機械指數的確定方法或NEMA UD 2-2004 聲學輸出測量標準
[不符合]
未檢附本案擬申請主機及本案所搭配探頭之超音波設備的特殊安全性基本要求(Particular requirements for the basic safety, IEC 60601-2-37)測試評估報告

[3]生物相容性評估(所有探頭)
	生物相容ISO 10993-1。
[不符合]
未檢附擬申請探頭生物相容性評估報告
。
[4]軟體驗證報告
OK，檢附Software Validation Report。

[5]網路安全評估報告
OK，檢附Cybersecurity Self-Assessment Report。
[6]特殊軟體功能測試評估(量測功能)
[不符合]
未檢附本案所有臨床可使用(包含所有中文說明書記載之測量(臨床項目) 項目。

[7]AI CADe/CADx軟體文件
[不符合]
未檢附本案基於AI技術具有電腦輔助偵測/診斷(CADe/CADx)軟體功能之技術資料(請參考《人工智慧/機器學習技術之電腦輔助偵測(CADe)及電腦輔助診斷(CADx)醫療器材查驗登記技術指引》。
[8]最終產品品質檢測
OK，檢附原廠品質檢驗報告

審查結果：

所檢附原廠技術資料不完整，請補充以下規格資料：
3.請檢附所有擬申請探頭之軸向解析度、側向解析度等規格資料。
4.請檢附本案所有臨床可使用(包含所有中文說明書記載之測量(臨床項目) 項目及其測量準確度等規格資料。
5. 請檢附本案具有定量/半定量量測能力之軟體功能及其測量準確度等規格資料。

5.性能測試資料：
請檢附本案擬申請探頭之功能性測試評估結果(符合探頭規格，如掃描角度、掃描長度、頻率範圍、軸向解析度、側向解析度。

