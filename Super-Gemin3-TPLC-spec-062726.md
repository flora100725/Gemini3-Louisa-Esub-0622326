## 1. Executive Summary & Architecture Paradigm

The AURA-7 platform is a next-generation Cloud-Native Regulatory & Compliance Hub purpose-built for the Taiwan Food and Drug Administration (TFDA). It converts fragmented, multi-lingual, and heterogenous medical device transactional logs into real-time, deterministic, and legally actionable oversight vectors.

```
                                  [ AURA-7 SYSTEM GRAPH ARCHITECTURE ]
                                  
  [Ingestion / Multi-Source Gateway]         [Deterministic Compute Core]        [Neural Orchestration Layer]
 ┌──────────────────────────────────┐      ┌──────────────────────────────┐    ┌──────────────────────────────┐
 │ • Raw Text Recalls / Multi-Docs  │      │ • DuckDB WebAssembly (WASM)  │    │ • Gemini Advanced Engine     │
 │ • Mixed CSV / TSV / Markdown     │ ===> │ • In-Browser Temporal DB     │ <=>│ • Prompt-to-SQL Compiler     │
 │ • Real-time API Telemetry Integr.│      │ • Relational Integrity Layer │    │ • Neural RAG Document Parser │
 └──────────────────────────────────┘      └──────────────┬───────────────┘    └──────────────────────────────┘
                                                          │
                                                          ▼
                                           [Visualization & Export Mesh]
                                           ┌──────────────────────────────┐
                                           │ • Real-time Interactive Grid │
                                           │ • D3 / WGS-84 Geospatial Canvas
                                           │ • Multi-format Transpilers   │
                                           └──────────────────────────────┘

```

### Architectural Principles & Runtime Specifications

* **The Engine Blueprint:** Zero-trust, offline-first client-side heavy micro-architecture, using an embedded relational execution matrix alongside asynchronous neural inference loops.
* **The UI Theme System (The "Wow" Visual Engine):** The platform features a high-performance visual layer with real-time UI/UX state management, supporting 10 Pantone operational modes, instant high-contrast switching, and advanced interactive animations without rendering lag.
* **Decoupled Dual-Core Compute Processing:**
* **Deterministic Core:** Run locally via an in-memory instance of **DuckDB WebAssembly (WASM)**. This environment hosts all imported, pasted, and default relational schemas, performing rapid analytical joins and analytical functions over thousands of rows directly inside the browser sandbox.
* **Probabilistic Core:** Driven by the **Gemini API** using structured JSON schemas. It handles unstructured text extraction, high-context regulatory reasoning, and prompt-to-SQL translation, turning natural language into verified SQL queries run against the local DuckDB database.



---

## 2. Global State & Unified Database Architecture

To facilitate seamless multi-dataset operations, data modification, text-to-SQL compilation, and direct schema linking across the five mandatory regulatory domains and the incoming document ingestion pipelines, the global state is managed within an enterprise-grade relational structure inside DuckDB WASM.

```
                              [ UNIFIED SCHEMA INTER-RELATIONAL MODEL ]
                              
  ┌──────────────────────────────┐             ┌──────────────────────────────┐
  │      tudid_dataset           │             │    device_license_dataset    │
  ├──────────────────────────────┤             ├──────────────────────────────┤
  │ PK   id (Autoincrement)      │             │ PK   license_number (VARCHAR)│<───┐
  │ FK   license_number  ────────┼────────────>│      device_class (VARCHAR)  │    │
  │      udi_issuing_org         │             │      chinese_name (VARCHAR)  │    │
  │      primary_di              │             │      category_one (VARCHAR)  │    │
  │      model_number            │             │      applicant_name (VARCHAR)│    │
  └──────────────────────────────┘             │      expiry_date (DATE)      │    │
                                               └──────────────────────────────┘    │
  ┌──────────────────────────────┐                                                 │
  │      recall_dataset          │             ┌──────────────────────────────┐    │
  ├──────────────────────────────┤             │         qms_dataset          │    │
  │ PK   id (Autoincrement)      │             ├──────────────────────────────┤    │
  │      title (VARCHAR)         │             │ PK   qsd_number (VARCHAR)    │    │
  │      device_name_recall      │             │      manufacturer_name       │    │
  │      manufacturer (VARCHAR)  │             │      country (VARCHAR)       │    │
  │      recall_date (DATE)      │             │      applicant_company       │    │
  │      recall_class (VARCHAR)  │             └──────────────────────────────┘    │
  │      udi (VARCHAR)           │                                                 │
  │      sn_lot_no (VARCHAR)     │             ┌──────────────────────────────┐    │
  │      reason (TEXT)           │             │     who_medevis_dataset      │    │
  └──────────────────────────────┘             ├──────────────────────────────┤    │
                                               │ PK   id (Autoincrement)      │    │
                                               │      device_name (VARCHAR)   │    │
                                               │      emdn_code (VARCHAR)     │    │
                                               │      emdn_term (VARCHAR)     │    │
                                               │      gmdn_code (VARCHAR)     │    │
                                               └──────────────────────────────┘    │
                                                                                   │
  ┌──────────────────────────────┐                                                 │
  │     recall_doc_ingestion     │                                                 │
  ├──────────────────────────────┤                                                 │
  │ PK   doc_id (UUID)           │                                                 │
  │      raw_content (TEXT)      │                                                 │
  │      extraction_status       │                                                 │
  │ FK   associated_license ─────┼─────────────────────────────────────────────────┘
  └──────────────────────────────┘

```

### Table Definitions & Exact Typed Relational Schemas

#### 1. Medical Device Recall Dataset (`recall_dataset`)

```sql
CREATE TABLE recall_dataset (
    id INTEGER PRIMARY KEY DEFAULT nextval('seq_recall_id'),
    title VARCHAR NOT NULL,
    device_name_for_recall VARCHAR NOT NULL,
    manufacturer VARCHAR NOT NULL,
    recall_date DATE,
    recall_class VARCHAR(10),
    udi VARCHAR(50),
    sn_lot_no VARCHAR(100),
    reason_for_recall TEXT,
    is_default BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

#### 2. Medical Device License Dataset (`device_license_dataset`)

```sql
CREATE TABLE device_license_dataset (
    license_number VARCHAR PRIMARY KEY, -- 許可證字號
    device_class VARCHAR(10),           -- 醫療器材級數
    chinese_name VARCHAR NOT NULL,      -- 中文品名
    category_one VARCHAR,               -- 醫器次類別一
    applicant_name VARCHAR,             -- 申請商名稱
    manufacturer_country VARCHAR(10),  -- 製造廠國別
    expiry_date DATE,                   -- 有效日期
    classification_code VARCHAR(20),   -- 分類分級代碼
    applicant_tax_id VARCHAR(20),       -- 申請商統一編號
    is_default BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

#### 3. Taiwan Unique Device Identifier Database Dataset (`tudid_dataset`)

```sql
CREATE TABLE tudid_dataset (
    id INTEGER PRIMARY KEY DEFAULT nextval('seq_tudid_id'),
    license_number_type VARCHAR,        -- 許可證字號（類型）
    udi_issuing_org VARCHAR(20),        -- UDI發碼機構
    primary_di VARCHAR(50) NOT NULL,    -- 基本DI
    model_number VARCHAR(100),          -- 型號
    chinese_name VARCHAR,               -- 中文品名
    revocation_status VARCHAR(20),      -- 註銷狀態
    device_class VARCHAR(10),           -- 醫療器材級數
    applicant_name VARCHAR,             -- 申請商名稱
    category_one VARCHAR,               -- 醫器次類別一
    is_default BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

#### 4. WHO Medevis Dataset (`who_medevis_dataset`)

```sql
CREATE TABLE who_medevis_dataset (
    id INTEGER PRIMARY KEY DEFAULT nextval('seq_who_medevis_id'),
    device_name VARCHAR NOT NULL,
    emdn_code VARCHAR(50),
    emdn_term TEXT,
    gmdn_code VARCHAR(50),
    gmdn_term TEXT,
    is_default BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

#### 5. Quality Management System Dataset (`qms_dataset`)

```sql
CREATE TABLE qms_dataset (
    id INTEGER PRIMARY KEY DEFAULT nextval('seq_qms_id'),
    manufacturer_name VARCHAR NOT NULL, -- 製造廠名稱
    manufacturer_country VARCHAR(10),  -- 製造廠國別
    manufacturer_address TEXT,          -- 製造廠地址
    applicant_company VARCHAR,          -- 藥商名稱
    qsd_number VARCHAR(50) NOT NULL,    -- QSD No
    expiry_date DATE,                   -- 有效日期
    case_status VARCHAR(50),            -- 案件狀況
    english_product_scope TEXT,         -- 英文品項名稱
    is_default BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

---

## 3. High-Fidelity UI/UX & Workflow Engineering

The interface is built to provide an intuitive, high-performance user experience, using standard layout patterns to structure operations and support long working sessions.

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│  AURA-7 SYSTEM HEADS-UP DISPLAY (HUD) TOP NAVIGATION CONTROLLER                                         │
│  [🌐 EN/繁中]  [🎨 PANTONE SELECTOR: Living Coral]  [🌗 DARK/LIGHT]  [🛰️ NETWORK LINK: TFDA SECURE VIPER] │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────┐ ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │ DATASET INTEGRATION  │ │ EXPLORATIVE SEARCH RADAR MESH                                            │  │
│  │ ├────────────────────┤ │ ├────────────────────────────────────────────────────────────────────────┤  │
│  │ │ • Recalls (MD/JSON)│ │ │ User Query: "Show Class 1 items expiring after 2026"                   │  │
│  │ │ • Licenses (CSV)   │ │ │ [ AI Agent prompt-to-SQL Compiler Action Processing... ]               │  │
│  │ │ • TUDID Base (CSV) │ │ ├────────────────────────────────────────────────────────────────────────┤  │
│  │ │ • WHO Medevis (TSV)│ │ │ SELECT * FROM device_license_dataset WHERE device_class = '3' ...     │  │
│  │ │ • QMS Audit (CSV)  │ │ └────────────────────────────────────────────────────────────────────────┘  │
│  │ └────────────────────┘ ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │ ┌────────────────────┐ │ HIGH-PERFORMANCE ANALYTICAL GRAPH COMPONENT                              │  │
│  │ │ DATA EXTRACTION    │ │ ├────────────────────────────────────────────────────────────────────────┤  │
│  │ │ ┌────────────────┐ │ │ [Interactive Multi-Axis Charting Matrix: Violations vs Expirations]    │  │
│  │ │ │ Drop Docs Here │ │ │                                                                          │  │
│  │ │ └────────────────┘ │ │ 📊 ████████████ 88%  📉 ████████ 43%  📈 ████████████████ 100%           │  │
│  │ └────────────────────┘ └──────────────────────────────────────────────────────────────────────────┘  │
│  └──────────────────────┘ ┌──────────────────────────────────────────────────────────────────────────┐  │
│                           │ REAL-TIME DATAGRID QUERY MATRIX EXPLORER                                 │  │
│                           │ ├────────────────────────────────────────────────────────────────────────┤  │
│                           │ │ [ ] LICENSE ID  │ NAME              │ RISK SCALE │ ACTIONS             │  │
│                           │ │ [x] TW-R-000508 │ Chinta Syringe    │ Class II   │ [Analyze] [Edit]    │  │
│                           │ │ [ ] TW-R-025432 │ Medtronic ICD     │ Class III  │ [Analyze] [Edit]    │  │
│                           │ └────────────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘

```

### The 10 Pantone Dynamic Workspace Theme Spec

The interface includes 10 Pantone design choices. Each selection dynamically shifts the UI's look, utilizing absolute values to maintain accessible contrast boundaries:

| # | Pantone Theme Name | Hex Code Primary | Hex Code Accent | Purpose / Contrast Target |
| --- | --- | --- | --- | --- |
| 1 | **Living Coral** (16-1546) | `#FF6F61` | `#4A4A4A` | Default State. High structural visibility for quick action. |
| 2 | **Classic Blue** (19-4052) | `#0F4C81` | `#F5F5F7` | Deep analytical profile. Minimizes eye strain during large logs audits. |
| 3 | **Ultimate Gray** (17-5104) | `#939597` | `#F5DF4D` | Tactical balance. Excellent for textual reconciliation tasks. |
| 4 | **Very Peri** (17-3938) | `#6667AB` | `#FFFFFF` | Modern creative interface for deep pattern matching discovery loops. |
| 5 | **Emerald** (17-5641) | `#009B77` | `#1A1A1A` | System Clean / Restitution mode. Optimal under extreme glare. |
| 6 | **Peach Fuzz** (13-1023) | `#FFBE98` | `#333333` | Reduced-luminance setting for long evening shifts. |
| 7 | **Radiant Orchid** (18-3224) | `#B565A7` | `#F0F0F0` | High-chroma focus matrix used during critical product containment. |
| 8 | **Illuminating** (13-0647) | `#F5DF4D` | `#111111` | Maximum Contrast alerting mode for rapid field interaction. |
| 9 | **Marsala** (18-1438) | `#955251` | `#FFFFFF` | Dark earthy background configuration tailored for low-lit labs. |
| 10 | **Tangerine Tango** (17-1463) | `#DD4124` | `#EFEFEF` | Urgent Action interface for real-time recall interventions. |

---

## 4. Multi-Source Dataset Integration Engine & Serialization Pipeline

The subsystem manages multiple input methods—direct text pasting, file uploads, and bulk markdown conversions—and saves datasets directly as `.md` files to ensure predictable default loads.

```
                   [ SUB-SYSTEM INGESTION AND SERIALIZATION WORKFLOW ]
                   
  [User Raw Input Interface]             [Hydration Matrix Parser]             [DuckDB State Persistence]
 ┌───────────────────────────┐         ┌───────────────────────────┐         ┌───────────────────────────┐
 │ • Raw Text Pasting Box    │         │ • JSON Token Sanitizer    │         │ • WASM Relational Hydrat. │
 │ • Drag-and-Drop Uploader  │ =======>│ • Character Set Normalizer│ =======>│ • Default Base Injection  │
 │ • System Local File Picker│         │ • Column Mapping Extractor│         │ • Live Index Creation     │
 └───────────────────────────┘         └─────────────┬─────────────┘         └───────────────────────────┘
                                                     │
                                                     ▼
                                      [Markdown Serialization Stream]
                                       ┌───────────────────────────┐
                                       │ • Front-matter Generation │
                                       │ • Structural Pipe Table   │
                                       │ • Base64 Compression Wrap │
                                       └───────────────────────────┘

```

### Raw Dataset Declarations & Static Compilations

#### Dataset 1: Medical Device Recall Base Dataset (`recall_dataset.md`)

```markdown
---
table_name: recall_dataset
records_count: 10
checksum: sha256_e836bfa791c
---

| title | device_name_for_recall | manufacturer | date | recall_class | udi | sn_lot_no | reason_for_recall |
|---|---|---|---|---|---|---|---|
| Class 2 Device Recall Intera Oncology | INTERA 3000 Hepatic Artery Infusion Pump, AP-0300H | Boston Scientific Corporation | 06/25/2026 | 2 | 00850014110147 | 20175 | Potential for leakage along the drug pathway from the pump through the end of the catheter. |
| Class 1 Device Recall Abiomed Impella CP Set with SmartAssist (10th Generation) | Impella CP Set with SmartAssist (10th Generation) containing affected 14Fr Low Profile Introducer Kit | Abiomed, Inc. | 06/25/2026 | 1 | 00813502013467 | Batch Number: 2041530 | Potential for thrombus (blood clot) formation during prolonged use of the introducer sheath. |
| Class 1 Device Recall Abiomed 14 Fr x 25 cm Low Profile Introducer Kit | Abiomed 14 Fr x 25 cm Low Profile Introducer Kit for Impella CP (Product Code: 1000435) | Abiomed, Inc. | 06/25/2026 | 1 | Not Explicitly Detailed in Log | Affected Product Batches under Code 1000435 | Potential for thrombus (blood clot) formation during prolonged use of the introducer sheath. |
| Class 1 Device Recall Abiomed 14 Fr x 13 cm Low Profile Introducer Kit | Abiomed 14 Fr x 13 cm Low Profile Introducer Kit for Impella CP (Product Code: 1000434) | Abiomed, Inc. | 06/25/2026 | 1 | Not Explicitly Detailed in Log | Affected Product Batches under Code 1000434 | Potential for thrombus (blood clot) formation during prolonged use of the introducer sheath. |
| Class 1 Device Recall Abiomed Impella CP Set with SmartAssist (Variant Entry) | Impella CP Set with SmartAssist (10th Generation) containing affected 14Fr Low Profile Introducer Kits | Abiomed, Inc. | 06/25/2026 | 1 | 00813502013467 | Batch Number: 2041530 | Potential for thrombus (blood clot) formation during prolonged use of the introducer sheath. |
| Class 2 Device Recall Medline Convenience Kits (Craniotomy Pack) | Convenience kits containing select SKUs of 10mL Polycarbonate Colored Syringes (CRANIOTOMY PACK-L) | Medline Industries, LP | 06/25/2026 | 2 | 10889942153497 | Affected Medline Kit Lots | Unapproved design changes made to the 10mL Polycarbonate Colored Syringes component outside of the FDA-cleared 510(k). |
| Class 2 Device Recall Medline Convenience Kits (Major Basin Set) | Convenience kits containing select SKUs of 10mL Polycarbonate Colored Syringes (MAJOR BASIN SET) | Medline Industries, LP | 06/25/2026 | 2 | Multiple UDI-DIs | Affected Medline Kit Lots | Unapproved design changes made to the 10mL Polycarbonate Colored Syringes component outside of the FDA-cleared 510(k). |
| Class 2 Device Recall Medline Convenience Kits (Kit Neuro Cstm) | Convenience kits containing select SKUs of 10mL Polycarbonate Colored Syringes (KIT NEURO CSTM) | Medline Industries, LP | 06/25/2026 | 2 | Multiple UDI-DIs | Affected Medline Kit Lots | Unapproved design changes made to the 10mL Polycarbonate Colored Syringes component outside of the FDA-cleared 510(k). |
| Class 2 Device Recall Medline Convenience Kits (Cataract Full Body) | Convenience kits containing select SKUs of 10mL Polycarbonate Colored Syringes (CATARACT FULL BOD) | Medline Industries, LP | 06/25/2026 | 2 | Multiple UDI-DIs | Affected Medline Kit Lots | Unapproved design changes made to the 10mL Polycarbonate Colored Syringes component outside of the FDA-cleared 510(k). |
| Class 2 Device Recall Medline Convenience Kits (Angio Pack Dyn) | Convenience kits containing select SKUs of 10mL Polycarbonate Colored Syringes (ANGIO PACK DYN) | Medline Industries, LP | 06/25/2026 | 2 | Multiple UDI-DIs | Affected Medline Kit Lots | Unapproved design changes made to the 10mL Polycarbonate Colored Syringes component outside of the FDA-cleared 510(k). |

```

#### Dataset 2: Medical Device License Base Dataset (`device_license_dataset.md`)

```markdown
---
table_name: device_license_dataset
records_count: 8
checksum: sha256_d10aef78391
---

| license_number | device_class | chinese_name | category_one | applicant_name | manufacturer_country | expiry_date | classification_code | applicant_tax_id |
|---|---|---|---|---|---|---|---|---|
| 衛部罕醫器輸字第000001號 | 2 | “沛佳”法斯樂-杜瓦伸縮式髓內釘系統 | N.3020 骨髓內固定桿 | 裕強生技股份有限公司 | CA | 2029-06-10 | N.3020 | 22365756 |
| 衛部醫器陸輸字第000506號 | 2 | “康德萊” 胰島素筆配套用針 | J.5570 皮下單腔針 | 台灣康翼股份有限公司 | CN | 2028-08-16 | J.5570 | 68213331 |
| 衛部醫器陸輸字第000507號 | 2 | 優盛電子體溫計 | J.2910 臨床電子體溫計 | 優盛醫學科技股份有限公司 | CN | 2018-09-05 | J.2910 | 23123644 |
| 衛部醫器陸輸字第000508號 | 2 | "勤達"一次性使用無菌注射器帶針/不帶針 | J.5860 活塞式注射筒 | 勤達醫療器材股份有限公司 | CN | 2028-09-12 | J.5860 | 80403820 |
| 衛部醫器陸輸字第000509號 | 2 | “帝寶” 紅外線耳溫槍 | J.2910 臨床電子體溫計 | 合世生醫科技股份有限公司 | CN | 2023-09-17 | J.2910 | 97304042 |
| 衛部醫器陸輸字第000510號 | 2 | “華爾怡”一次性使用全自動回縮型注射器(帶針) | J.5860 活塞式注射筒 | 佳醫健康事業股份有限公司 | CN | 2018-12-13 | J.5860 | 22854522 |
| 衛部醫器陸輸字第000521號 | 2 | 醫佳一次性使用真空採血管(無添加) | A.1675 血液樣本收集設備 | 日溢健康有限公司 | CN | 2018-09-30 | A.1675 | 28513656 |
| 衛部醫器陸輸字第000522號 | 2 | 嘉鎂一次性使用人體靜脈血樣採集容器(普通管) | A.1675 血液樣本收集設備 | 台灣嘉鎂聯合有限公司 | CN | 2018-11-13 | A.1675 | 25097092 |

```

#### Dataset 3: Taiwan Unique Device Identifier Database (`tudid_dataset.md`)

```markdown
---
table_name: tudid_dataset
records_count: 10
checksum: sha256_b394182af9d
---

| license_number_type | udi_issuing_org | primary_di | model_number | chinese_name | revocation_status | device_class | applicant_name | category_one |
|---|---|---|---|---|---|---|---|---|
| 衛部醫器陸輸字第000858號 | GS1 | 06944904054537 | 009-004766-00 | “邁瑞”生理監視器 |  | 2 | 博宣寧股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器陸輸字第000858號 | GS1 | 06944904054520 | 009-004765-00 | “邁瑞”生理監視器 |  | 2 | 博宣寧股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器陸輸字第000858號 | GS1 | 06944904054544 | 009-004771-00 | “邁瑞”生理監視器 |  | 2 | 博宣寧股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器陸輸字第000858號 | GS1 | 06944904054551 | 009-004772-00 | “邁瑞”生理監視器 |  | 2 | 博宣寧股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器陸輸字第000858號 | GS1 | 06944904054568 | 009-004777-00 | “邁瑞”生理監視器 |  | 2 | 博宣寧股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器陸輸字第000858號 | GS1 | 06944904054700 | 009-004782-00 | “邁瑞”生理監視器 |  | 2 | 博宣寧股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器陸輸字第000867號 | GS1 | 00884838117297 | 989803117297 | “飛利浦”心電圖儀 |  | 2 | 台灣飛利浦股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器陸輸字第000867號 | GS1 | 00884838117464 | 989803117464 | “飛利浦”心電圖儀 |  | 2 | 台灣飛利浦股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器製字第005325號 | GS1 | 04713696848073 | ecg102D | QOCA隨身心電圖量測儀 |  | 2 | 廣達電腦股份有限公司 | E.2340 心電圖描記器 |
| 衛部醫器輸字第025432號 | GS1 | 00643169018037 | DDBC3D4 | “美敦力”艾維拉植入式心臟整流去顫器 |  | 3 | 美敦力醫療產品股份有限公司 | E.3610 植入式心律器之脈搏產生器 |

```

#### Dataset 4: WHO Medevis Dataset (`who_medevis_dataset.md`)

```markdown
---
table_name: who_medevis_dataset
records_count: 5
checksum: sha256_319ffba2940
---

| device_name | emdn_code | emdn_term | gmdn_code | gmdn_term |
|---|---|---|---|---|
| Absorbent tipped applicator | M0499 | SPECIAL DRESSINGS - OTHER | 33722 | General-purpose absorbent tip applicator/swab, single-use |
| Absorbents, incontinence | T040101 | INCONTINENCE NAPPIES | 61850 | Absorbent underpad, non-antimicrobial, single-use |
| Access port, multi-instrument, laparoscopy | K010101 | TROCAR, SINGLE-USE | 38456 | Laparoscopic multi-instrument access port, single-use |
| Acetic acid | D050101 | PERACETIC AND ACETIC ACID WITH HYDROGEN PEROXIDE FOR THE DISINFECTION OF MEDICAL DEVICES | 47631 | Medical device disinfection agent |
| Airway, laryngeal | R010202 | LARYNGEAL TUBES | 45036 | Laryngeal mask airway, single-use |

```

#### Dataset 5: Quality Management System Dataset (`qms_dataset.md`)

```markdown
---
table_name: qms_dataset
records_count: 4
checksum: sha256_fa23948bf81
---

| manufacturer_name | manufacturer_country | manufacturer_address | applicant_company | qsd_number | expiry_date | case_status | english_product_scope |
|---|---|---|---|---|---|---|---|
| Jiangyu Health Innovation Medical Technology Chengdu Co., Ltd. | CHN | Rm 202, Bld 2, No.618, Fenghuang Road, Shuangliu Zone, China (Si-chuan) Chengdu Tianfu international biological city | 疆域醫創科技有限公司 | QSD16050 | 2026-06-02 | 准予核備 | Multi-parameter Detector manufacturing |
| Ray Co., Ltd. | KOR | 1F-3F, 4F(Part), 5F, 265, Daeji-ro, Suji-gu, Yongin-si, Gyeonggi-do, 16882 | 台灣瑞麗醫療器材股份有限公司 | QSD14667 | 2026-07-03 | 續予核備 | Medical image management and processing system manufacture, pack, label |
| RTD | FRA | 3 Rue Louis Neel Technoparc-Espace Gavaniere 38120 Saint-Egreve | 資生國際有限公司 | QSD8052 | 2026-07-10 | 續予核備 | Intraoral dental drill, Tooth shade resin material, Root canal post |
| Jiangsu Bonss Medical Technology Co., Ltd. | CHN | Building #7, No. 898, China Medical City Avenue, Hailing District, Taizhou City, 225316 | 奇裕企業股份有限公司 | QSD13384 | 2026-07-13 | 續予核備 | Electrosurgical cutting and coagulation device and accessories |

```

### Ingestion Interface Mechanics (Pasting, Uploading, Downloading)

The hydration workflow parses custom files using unified delimiters (`\n`, `\r\n`), matches headers against schema properties, and filters row values to fit specific column parameters.

```
[Markdown Pipe File Input] ──> [Front-matter Meta Stripper] ──> [Markdown Table Parser] ──> [DuckDB Cache Sync]

```

---

## 5. Conversational AI Text-to-SQL Compiler & Search Interface

The dynamic text-to-SQL compiler turns conversational text into structured relational commands, running them against the browser-based client datastore via a dedicated API mapping.

```
                                  [ CONVERSATIONAL SYNTAX RECONSTRUCTION ]
                                  
  [User Conversational String]           [Gemini Core Translation]             [DuckDB Local Compilation]
 ┌───────────────────────────┐         ┌───────────────────────────┐         ┌───────────────────────────┐
 │ "Find all Class 1 items   │         │ • System Schema Injection │         │ • Pre-Execution Safety    │
 │  recalled due to thrombus │ =======>│ • Strict AST Generation   │ =======>│   Token Validation Check  │
 │  risks during 2026."      │         │ • JSON Execution Spec     │         │ • Dynamic Execution Plan  │
 └───────────────────────────┘         └─────────────┬─────────────┘         └───────────────────────────┘
                                                     │
                                                     ▼
                                         [Compiled Executable SQL]
                                        ┌───────────────────────────┐
                                        │ SELECT * FROM recall_table│
                                        │ WHERE recall_class = '1'  │
                                        │ AND reason LIKE '%thromb%'│
                                        └───────────────────────────┘

```

### Contextual Intent Assembly

```
You are an expert Text-to-SQL execution engine running over an in-memory browser-side DuckDB database containing high-stakes TFDA regulatory datasets.
Your task is to take a natural language user query and transform it into a mathematically sound, performant, and safe SQL statement.

[AVAILABLE REGULATORY TARGET TABLES & STORAGE FIELDS]
1. recall_dataset (fields: id, title, device_name_for_recall, manufacturer, recall_date, recall_class, udi, sn_lot_no, reason_for_recall)
2. device_license_dataset (fields: license_number, device_class, chinese_name, category_one, applicant_name, manufacturer_country, expiry_date, classification_code, applicant_tax_id)
3. tudid_dataset (fields: id, license_number_type, udi_issuing_org, primary_di, model_number, chinese_name, revocation_status, device_class, applicant_name, category_one)
4. who_medevis_dataset (fields: id, device_name, emdn_code, emdn_term, gmdn_code, gmdn_term)
5. qms_dataset (fields: id, manufacturer_name, manufacturer_country, manufacturer_address, applicant_company, qsd_number, expiry_date, case_status, english_product_scope)

[COMPILATION & SAFETY EXECUTION CONSTRAINTS]
- Output absolute JSON format conforming strictly to the format schema: {"sql_command": "SELECT ...", "explanation": "..."}.
- Do NOT alter data or issue destructive commands (DROP, DELETE, UPDATE, INSERT, ALTER).
- If the text requests modifications, output a safe read-only SELECT command tracking those target properties.
- Use ILIKE queries for loose textual substring matching. Handle traditional and simplified Chinese strings correctly.
- Always match date comparisons against the ISO date structure ('YYYY-MM-DD').

```

### Prompt-to-SQL Multi-Dataset Join Target Scenarios

#### Scenario A: Cross-referencing Recalled Components with Active QMS Systems

* *User Prompt:* "Show me all device recalls where the manufacturer's quality system is certified under a current QSD number from China (CHN), listing reasons and expiration dates."
* *Generated Compiled SQL:*

```sql
SELECT r.title, r.device_name_for_recall, q.qsd_number, q.manufacturer_name, q.expiry_date, r.reason_for_recall
FROM recall_dataset r
JOIN qms_dataset q ON r.manufacturer = q.manufacturer_name
WHERE q.manufacturer_country = 'CHN' AND q.expiry_date >= CAST('2026-01-01' AS DATE)
ORDER BY q.expiry_date ASC;

```

#### Scenario B: Tracking Global WHO Classifications Against Active Local Licenses

* *User Prompt:* "Find all domestic medical devices containing active licenses that overlap with WHO Medevis general-purpose applicator definitions using EMDN category codes."
* *Generated Compiled SQL:*

```sql
SELECT dl.license_number, dl.chinese_name, dl.applicant_name, wm.device_name, wm.emdn_code, wm.gmdn_code
FROM device_license_dataset dl
JOIN who_medevis_dataset wm ON dl.chinese_name ILIKE CONCAT('%', wm.device_name, '%')
OR dl.category_one ILIKE CONCAT('%', wm.device_name, '%')
WHERE wm.emdn_code = 'M0499';

```

---

## 6. Analytical Dashboard & Export Multi-Format Matrix

The system includes a dedicated visual layout for exploring search results, featuring clear data displays and multi-format download capabilities.

### Multi-Format Export Methods

* **CSV:** Streamed via a dynamic browser anchor element using a runtime-generated Blob object (`data:text/csv;charset=utf-8,\uFEFF`).
* **Markdown (.md):** Compiled into standard Markdown pipe-delimited tables, complete with system audit headers and table attributes.
* **PDF:** Rendered into structural layouts optimized for print layout constraints via standard CSS `@media print` directives.
* **Google Sheets:** Extracted into tabular arrays and formatted into serverless payload batches compatible with the Google Sheets API endpoint.

---

## 7. Advanced Intelligent AI Features Engine (The "Wow" AI Layer)

The application features three specialized AI engines built to run automated diagnostics on medical device records and cross-verify registry listings.

```
                              [ TRIPLE AI ENG-ENGINE CORNERSTONE MESH ]
                              
  ┌──────────────────────────────┐  ┌──────────────────────────────┐  ┌──────────────────────────────┐
  │   1. TRANS-BORDER LITIGATION │  │  2. KINETIC DECAY ESTIMATOR  │  │  3. HEURISTIC LEDGER HUNTER  │
  ├──────────────────────────────┤  ├──────────────────────────────┤  ├──────────────────────────────┤
  │ Analyzes conflicting manufacturer│  │ Evaluates real-time risk index │  │ Flags unusual transaction    │
  │ and hospital records using   │  │ using shelf-life safety      │  │ patterns or discrepancies in │
  │ legal precedent models.      │  │ decay equations.             │  │ registry data automatically. │
  └──────────────────────────────┘  └──────────────────────────────┘  └──────────────────────────────┘

```

### Feature 1: Multi-Party Registry Dispute & Conflict Resolution Engine

* **Purpose:** Resolves reporting mismatches when a manufacturer claims a device was shipped but the hospital record shows no receipt.
* **Mathematical Processing & Analysis Logic:** Uses deep logic trees to check supply records against the tracking requirements of the Medical Device Act. The AI flags filing timeline gaps and produces a pre-formatted legal notice detailing accountability.

### Feature 2: Predictive Path Expiry & Kinetic Quality Decay Estimator

* **Purpose:** Estimates the remaining safe shelf life of devices held in regional warehouses during logistics delays or temperature issues.
* **Mathematical Processing & Analysis Logic:** Uses basic degradation kinetics to estimate device stability over time:

$$\text{Stability} = \kappa \cdot e^{-\alpha \cdot \Delta t}$$

where $\kappa$ represents the initial material coefficient, $\alpha$ is the environmental wear rate, and $\Delta t$ tracks the transit time gap. The agent calculates this index to suggest moving short-dated stock to high-volume medical facilities.

### Feature 3: Self-Directed Heuristic Ledger Anomaly Hunter

* **Purpose:** Scans the database to find unusual records, unlogged entries, or unapproved product modifications without manual prompt engineering.
* **Mathematical Processing & Analysis Logic:** Uses a weighted multi-factor scoring grid to evaluate risk levels across records:

$$\text{Risk Score} = w_1 \cdot C_{\text{class}} + w_2 \cdot T_{\text{delay}} + w_3 \cdot M_{\text{mismatch}}$$

The engine flags items with high risk scores directly on the main dashboard to draw immediate attention to potential issues.

---

## 8. Unstructured Recall Document to JSON Structural Ingestion Module

This specialized data module processes raw text, unstructured recall notices, or unformatted logs and converts them into standardized JSON data structures.

```
                    [ UNSTRUCTURED TEXT TRANSFORM PARSING PIPELINE ]
                    
  [Raw Mixed Document Input]              [System Grammar Mapping]              [JSON Schema Compilation]
 ┌───────────────────────────┐         ┌───────────────────────────┐         ┌───────────────────────────┐
 │ "NOTICE: Abiomed recalls  │         │ • Entity Core Segmenter   │         │ • Strict Property Checker │
 │  Impella CP system due to │ =======>│ • Key-Value Pair Extractor│ =======>│ • JSON Array Compilator   │
 │  thrombus batch 2041530"  │         │ • Format Sanitizer Core   │         │ • Schema Validation Sync  │
 └───────────────────────────┘         └─────────────┬─────────────┘         └───────────────────────────┘
                                                     │
                                                     ▼
                                         [Standardized Clean JSON]
                                        ┌───────────────────────────┐
                                        │ {                         │
                                        │   "udi": "008135020...",  │
                                        │   "sn_lot_no": "2041530"  │
                                        │ }                         │
                                        └───────────────────────────┘

```

### Extraction Grammar & Validation Constraints

The system parses unstructured inputs into a verified schema using explicit extraction rules:

* Extract fields matching: `title`, `device_name_for_recall`, `manufacturer`, `date` (normalized to `MM/DD/YYYY`), `recall_class` (`1`, `2`, or `3`), `udi`, `sn_lot_no`, and `reason_for_recall`.
* If a field value is missing from the source text, explicitly populate it as `"Not Explicitly Detailed in Log"`. Do not generate placeholder values.

### Real-world Extraction Scenario & Interactive Mapping

#### Input Raw Document Stream Text

```text
ALERT RUNTIME LOG // TFDA EMERGENCY POSTING WRAPPER
An official action has been recorded on June 25, 2026. The firm Abiomed, Inc. has finalized an urgent declaration regarding their high-demand Impella CP Set with SmartAssist (10th Generation). This specific bundle is known to ship containing the affected 14Fr Low Profile Introducer Kit. 
Internal regulatory field monitoring teams have flagged this as a Class 1 level hazard scenario. The primary global identification barcoding record is registered under UDI 00813502013467. 
Physical product packages discovered in hospital inventories can be accurately verified by inspecting the production packaging printout indicating Batch Number: 2041530. 
The critical hazard context forcing this step points to an immediate Potential for thrombus (blood clot) formation during prolonged clinical use of the introducer sheath mechanism. Urgent containment is requested.

```

#### Compiled Output Target JSON Structure

```json
[
  {
    "title": "Class 1 Device Recall Abiomed Impella CP Set with SmartAssist (10th Generation)",
    "device_name_for_recall": "Impella CP Set with SmartAssist (10th Generation) containing affected 14Fr Low Profile Introducer Kit",
    "manufacturer": "Abiomed, Inc.",
    "date": "06/25/2026",
    "recall_class": "1",
    "udi": "00813502013467",
    "sn_lot_no": "Batch Number: 2041530",
    "reason_for_recall": "Potential for thrombus (blood clot) formation during prolonged use of the introducer sheath."
  }
]

```

---

## 9. Comprehensive System Risk Assessment & Proactive Bug Mitigation

To ensure reliable performance during multi-user operations and heavy data processing, the platform addresses potential technical issues with explicit fallback mechanisms.

### Performance & Processing Optimization

1. **Asynchronous Processing:** Large text extractions or complex multi-table queries run on background worker threads, keeping the main interface responsive.
2. **Local Token Pre-checks:** User prompts are validated locally before calling external APIs, ensuring query integrity and preventing connection overhead.
3. **Database Constraints:** Primary keys are validated during file ingestion to prevent record duplication and maintain database stability.

---

## 10. System Evolution & Regulatory Tracking Framework

To assist TFDA engineering teams with system maintenance and future development, the platform uses a structured tracking framework across four primary technical categories.

```
                              [ MAINTENANCE INTERROGATION RADAR ]
                              
  ┌──────────────────────────────┐  ┌──────────────────────────────┐
  │ 1. POLICY & LAW COHERENCE    │  │ 2. GEOSPATIAL DATA ACCURACY  │
  ├──────────────────────────────┤  ├──────────────────────────────┤
  │ Tracks medical device data   │  │ Validates coordinate values  │
  │ reporting intervals and raw  │  │ and checks geographic bounds │
  │ record retention policies.   │  │ for regional logging points. │
  └──────────────────────────────┘  └──────────────────────────────┘
  ┌──────────────────────────────┐  ┌──────────────────────────────┐
  │ 3. AGENT BEHAVIOR OPTIMIZATION│  │ 4. VISUAL ENGINE ACCURACY    │
  ├──────────────────────────────┤  ├──────────────────────────────┤
  │ Monitors query translation   │  │ Maintains layout readability │
  │ fidelity and prevents logic  │  │ metrics across all selected  │
  │ generation errors.           │  │ device theme settings.       │
  └──────────────────────────────┘  └──────────────────────────────┘

```

### 1. TFDA Policy & Legal Coherence

#### Q1: Device Lifecycle Monitoring Rules

Which high-risk device categories require immediate updates to tracking logs under Article 22 of the Medical Device Act, and what are the mandatory reporting timeframes for field updates?

#### Q2: Parallel Import Discovery Workflows

When the system flags an unrecorded serial number ("Ghost Stock"), what automated validation checks must be run against custom house manifest logs to verify potential parallel imports?

#### Q3: Regulatory Compliance Timelines

What is the standard correction window allowed for manufacturers to resolve structural data errors before the system generates an official TFDA non-compliance warning letter?

#### Q4: Cross-Institutional Data Sharing

What data classification rules govern the transfer of short-dated medical device inventory records between private medical networks and public hospital systems?

#### Q5: Digital Document Admissibility

Does an automated conflict resolution report generated by the system meet the evidentiary requirements for presentation in administrative court actions under the Taiwan Code of Administrative Procedure?

### 2. Geospatial Precision & Logistics Telemetry

#### Q6: Real-time Weather Integration

How are local weather warnings and infrastructure status data integrated into the route planning engine to update delivery timelines for remote clinics?

#### Q7: Offshore Island Logistics Management

What additional transit buffers are applied to tracking algorithms for offshore medical centers to account for weather-related maritime transport delays?

#### Q8: Cold Chain Temperature Monitoring

How does the system process real-time Bluetooth or cellular temperature logs to maintain continuous quality verification within device records?

#### Q9: Coordinate Verification Filters

What coordinate validation filters are applied during batch uploads to catch swapped latitude and longitude values before they affect regional map plotting?

#### Q10: Facility Micro-Mapping Integration

How can the interface scale from regional transport overviews down to facility-level tracking, enabling direct inventory verification within specific storage areas?

### 3. Agent Execution Fidelity & Knowledge Guardrails

#### Q11: Preventing Text-to-SQL Extraction Deviations

What architectural rules prevent the translation agent from outputting invalid SQL syntax or introducing logical deviations when translating complex multi-table queries?

#### Q12: API Rate Limiting Policies

What rate-limiting controls are active in the local communication layers to manage API request volume and maintain application stability under heavy load?

#### Q13: Multi-Factor Scoring Calibration

How are the internal weights adjusted within the Anomaly Hunter engine to balance minor tracking delays against major validation errors like unidentified serial numbers?

#### Q14: International Barcode Standards Compatibility

How does the layout logic handle shifts between US FDA global formats and EU MDR variable-length barcodes without breaking database parsing matrices?

#### Q15: Client-Side Data Masking

What data sanitization filters strip patient names and personal identification information from logs before sending records to processing layers?

### 4. Visual Interface Diagnostics & Operational Continuity

#### Q16: High-Volume Rendering Management

How does the interface handle display performance when active tracking records exceed ten thousand entries, avoiding interface lag or rendering delays?

#### Q17: Interface Contrast Verification

Do all ten selectable design themes maintain an absolute 4.5:1 contrast ratio across primary text elements to meet standard accessibility requirements for prolonged use?

#### Q18: Offline Storage Synchronization

What local storage mechanisms store structural modifications or entry adjustments when field teams operate in low-connectivity environments?

#### Q19: Print Layout Document Optimization

What print styles ensure that system summaries and generated markdown tables fit standard print formats neatly without cropping important columns?

#### Q20: Cryptographic Record Integrity

How are individual entry updates signed and verified within the local database instance to ensure that logs remain reliable and audit-ready?

---

## 11. Appendix: Execution Reference Matrix

To assist development teams during implementation, this quick-reference guide maps the system's operational flows to their respective database dependencies and processing modes.

```
┌───────────────────────────┬───────────────────────────┬───────────────────────────┐
│ Functional Domain         │ Core Database Dependency  │ Processing Mode           │
├───────────────────────────┼───────────────────────────┼───────────────────────────┤
│ Multi-Doc JSON Extraction │ `recall_dataset`          │ Probabilistic (Gemini)    │
│ Prompt-to-SQL Compiler    │ All Relational Tables     │ Hybrid (Gemini + DuckDB)  │
│ Geospatial Routing        │ `qms_dataset` / Stations  │ Deterministic (WASM Core) │
│ Ledger Anomaly Hunter     │ Full Database Mesh        │ Deterministic (WASM Core) │
└───────────────────────────┴───────────────────────────┴───────────────────────────┘

```

This technical blueprint provides a reliable foundation for implementing the AURA-7 platform, ensuring robust data management and efficient automated oversight workflows for the TFDA.
