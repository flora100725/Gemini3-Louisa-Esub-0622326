TECHNICAL DESIGN SPECIFICATION: 510(k) PRE-MARKET NOTIFICATION INTERACTIVE REGULATORY REVIEW AGENT (TAIWAN TFDA COMPLIANT)
Document Version: 4.2.0
Status: Approved for Architectural Implementation
Standard Framework Mapping: Taiwan Medical Device Act (醫療器材管理法), TFDA 醫療器材許可證核發與登錄及年度申報準則, ISO 13485:2016, IEC 62304, IEC 60601-1, IEC 60601-1-2
Target Audiences: Principal AI Architects, Regulatory Affairs Directors, Lead Full-Stack Engineers, Lead Systems Engineers
1. Executive Summary & Administrative Context
The compliance evaluation process for medical device registrations—specifically within the Traditional 510(k) pre-market notification pathway under Taiwan Food and Drug Administration (TFDA) jurisdiction—is a labor-intensive, precise undertaking. Reviewing authorities and Regulatory Affairs (RA) personnel must cross-examine massive technical dossiers against rigid standards. Small oversights, such as discrepancies in address abbreviating, lack of active post-market software protocols, or data leakage in Machine Learning models (SaMD), frequently result in costly, multi-month regulatory delays.
This document specifies the technical design, framework, and micro-component blueprints for a Wow-Tier Agentic 510(k) Regulatory Review Workbench. This system is architected as an offline-first, client-and-server-side application written in TypeScript with React, Vite, and tailwindcss. It utilizes large language model (LLM) agents (defaulting to gemini-3.5-flash or gemini-3.1-pro-preview) to perform deep semantics extraction, technical conformity evaluations, and heuristic validation.
Core Objectives of the Workspace:
Dossier & Skill Ingestion: Direct support for drag-and-drop or copy-pasted medical device submission dossiers (textually raw, structured PDFs, JSON payloads, or markdown) mapped against customizable core rule libraries (skill.md).
Interactive Workbench & State Retaining: A split-screen single-page application (SPA) layout allowing regulatory reviewers to verify eight critical chapters of TFDA registration, modify evaluations, capture notes, and view AI recommendations.
Advanced WOW Heuristics (6 Cognitive Systems): Real-time detection of corporate discrepancies, software version locked/adaptive parameters, test-dataset cross-contamination, automated TFDA pathway advice, statutory language mapping, and laboratory accreditation vetting.
Living Log & Cognitive Console: Real-time progress updates displaying step-by-step reasoning logs, tokens consumed, and current subsystem processing cycles.
Interactive Double-Tier Dashboard: A 6-graph analytical panel detailing structural compliance via Recharts visualizations.
Polished Multi-Format Exports: One-click generation of beautifully-formatted documents in Markdown, HTML, and Google Doc copy-scratchpads.
2. Global System Architecture
The application is structured as a full-stack Node.js container with an ultra-responsive React SPA on the frontend and an Express + Vite server on the backend. This architecture prevents API keys from leaking to client web inspectors, executes heavy text-matching processes, and maintains state in the local sandbox database (localStorage).
code
Code
┌─────────────────────────────────────────────────────────────┐
                              │                 User Local Machine Web Browser              │
                              │                                                             │
                              │    ┌──────────────────┐               ┌────────────────┐    │
                              │    │ Ingestion View   │               │   6-Graph      │    │
                              │    │ (Dossiers/Skills)│               │  Dashboard     │    │
                              │    └────────┬─────────┘               └────────▲───────┘    │
                              │             │                                  │            │
                              │             ▼                                  │            │
                              │    ┌──────────────────┐               ┌────────┴───────┐    │
                              │    │ Review Workbench │ ◄───────────► │ Live Cognitive │    │
                              │    │  & Accordion List│               │    Console     │    │
                              │    └────────┬─────────┘               └────────────────┘    │
                              │             │ Use State                                     │
                              └─────────────┼───────────────────────────────────────────────┘
                                            │ Updates
                                            ▼
                              ┌─────────────────────────────────────────────────────────────┐
                              │            Server-Side Express Proxy API Server             │
                              │                                                             │
                              │   ┌────────────────────┐            ┌────────────────────┐  │
                              │   │ Heuristic Parsing  │            │ Google GenAI Core  │  │
                              │   │ Engine (Fallback)  │ ◄────────► │ (gemini-3.5-flash) │  │
                              │   └────────────────────┘            └────────────────────┘  │
                              └─────────────────────────────────────────────────────────────┘
2.1 File System Structure and Key Roles
To preserve modularity, scalability, and type safety, the application is organized with strict separation of types, helper modules, and visual layouts:
/metadata.json: Controls the applet's permissions (cameras, frames, microphone) and declares major server-side API capabilities.
/package.json: Explicitly manages development scripts (dev via tsx, build via vite & esbuild, and CJS bundling settings).
/src/types.ts: Centralized source of truth for TypeScript typings. Defines TypeScript interfaces for review items, sections, live terminal logs, export entities, and dashboard parameters.
/src/data.ts: Houses the initial mock database representing Taiwan’s official 醫療器材查驗登記 checklists, demo submission materials, and the default skill.md regulatory guidelines.
/src/components/DashboardCharts.tsx: Modularized visual component managing Recharts implementations (Radar, Pie, Scatter, Bar, Area, and Time-series graphs).
/src/components/LiveConsole.tsx: Implements the terminal cognitive console with custom logs, process indicators, and real-time state alerts.
/src/App.tsx: Main entrypoint controller managing application states, file ingestions, API fetch triggers, export modalities, and page templates.
/server.ts: Express backend entrypoint managing secure Google GenAI SDK integrations, fallback heuristics engines, and production builds.
3. Data Schema & Comprehensive Types Specification
To prevent runtime errors and ensure strict contract alignments between the client workspace and the server, we define the following unified type model inside /src/types.ts:
code
TypeScript
export type ReviewStatus = 'pass' | 'fail' | 'na' | 'pending';

export interface ReviewItem {
  id: string;                  // Maps to TFDA standard clause code (e.g., "1-2", "8-6")
  text: string;                // Technical descriptive rules mandated by the regulatory code
  note: string;                // Editable review annotation entered by the regulatory authority
  status: ReviewStatus;        // Present status of conformity
  suggestedAction?: string;    // AI-generated corrective action or justification guidelines
  discrepancyMatch?: number;   // Calculated similarity score or discrepancy probability (0-100)
}

export interface ReviewSection {
  id: string;                  // Main chapter key (e.g., "sec1", "sec5")
  title: string;               // Traditional Chinese chapter header name
  items: ReviewItem[];         // List of inner checks with metadata
}

export interface LiveLog {
  id: string;                  // Distinct logger UUID
  timestamp: string;           // System ISO timestamp
  level: 'info' | 'warning' | 'success' | 'error';
  message: string;             // Informative terminal message
}

export interface AIAnalysisResult {
  stats: {
    pass: number;
    fail: number;
    na: number;
    pending: number;
  };
  reconciliationScore: number;  // Multi-agent address match similarity percentage
  contaminationThreat: number;  // Overlap metric representing testing leak risk
  adaptiveScore: number;        // ML algorithm learning vector (0 for locked, 1-100 for continuous)
  classificationPanel: string;  // Predicted TFDA committee assignment
  suggestedProductCode: string; // Recommended TFDA / FDA classification code (three-letter standard)
  regulatoryClass: string;      // TFDA risk category (Class I, II, III)
  chapterScores: {
    administrative: number;     // Subsystem score (0-100)
    qms: number;
    technical: number;
    labeling: number;
    biocompatibility: number;
    software: number;
  };
  addressDistances: Array<{
    dossier: string;            // Name of technical document (e.g., CFSM, Application Form)
    distance: number;           // Spelling similarity metric (0-100)
    address: string;            // Extracted text string representing address
  }>;
  contaminationScatter: Array<{
    id: number;
    trainingSimilarity: number; // Y-axis correlation
    testingSimilarity: number;  // X-axis correlation
    label: string;              // Diagnostic record metadata (Patient ID / Waveform segment)
    group: string;              // "Contaminated Leak" or "Secure Test Validation Set"
  }>;
  standardsCoverage: Array<{
    standard: string;           // Reference (e.g., IEC 60601-1, IEC 62304)
    score: number;              // Compliant coverage index
    required: boolean;          // Mandatory statutory standard flag
  }>;
  timelinePoints: Array<{
    turn: number;               // Audit lifecycle step index
    tokens: number;             // Model processing length
    confidence: number;         // Model probability output index
    accuracy: number;           // Correctness threshold metric
  }>;
  itemsUpdates: Record<string, {
    status: ReviewStatus;
    note: string;
    suggestedAction?: string;
    discrepancyMatch?: number;
  }>;
}
4. Advanced Wow AI Core Systems (All 6 Features)
The intelligence layer of the system is composed of six distinct Wow AI systems. Each system is designed to identify subtle non-compliant variables that typically escape human audit teams.
code
Code
┌─────────────────────────────────────────────────────────────────┐
       │                     INPUT 510(k) FILE DOSSIERS                  │
       └────────────────────────────────┬────────────────────────────────┘
                                        │
                                        ▼
             ┌─────────────────────────────────────────────────────┐
             │       WOW AI SYSTEM COGNITIVE HARNESS ENGINE        │
             │                                                     │
             │  ┌───────────────────────┐   ┌───────────────────┐  │
             │  │ 1. Address Mismatch   │   │ 2. Locked vs      │  │
             │  │    Reconciliation     │   │    Adaptive Audit │  │
             │  └───────────────────────┘   └───────────────────┘  │
             │  ┌───────────────────────┐   ┌───────────────────┐  │
             │  │ 3. Data Contamination │   │ 4. Regulatory     │  │
             │  │    Detector           │   │    Pathway Advisor│  │
             │  └───────────────────────┘   └───────────────────┘  │
             │  ┌───────────────────────┐   ┌───────────────────┐  │
             │  │ 5. Legislative Draft  │   │ 6. Labs Accredit  │  │
             │  │    Synthesis          │   │    Audit          │  │
             │  └───────────────────────┘   └───────────────────┘  │
             └──────────────────────────┬──────────────────────────┘
                                        │
                                        ▼
       ┌─────────────────────────────────────────────────────────────────┐
       │                COGNITIVE MAPS TO 8 CHAPTERS STATE               │
       └─────────────────────────────────────────────────────────────────┘
4.1 System 1: Multi-Agent Discrepancy Reconciliation Engine
Objective
Validates spelling variations across administrative paperwork. In regulatory applications, even a minor spelling difference in a corporate address (e.g., "100 Innovation Way, Suite 401" vs. "100 Innovation Way, 4th Floor Room 401") across different submission documents can lead to bureaucratic delays or immediate rejection by TFDA clerks.
Mechanism
Named Entity Extraction: Extracts all instances of commercial names, manufacturing entities, assembly centers, and corporate addresses from the application form, Certificate of Free Sale (製售證明), and QMS/QSD registration.
Text Standardization: Normalizes typical address components (converting abbreviations like Suite → STE, Room → RM, 4th Floor → 4F, Boulevard → Blvd).
Levenshtein-Jaro-Winkler Distance Mapping: Computes mathematical character similarities and spatial text offsets between normalized addresses:
Interactive Action: When a similarity score is lower than a 95% threshold, the system flags the relevant address inputs, registers a warning status, calculates the mismatch, and recommends either a correction to the application form or a supporting letter of manufacturer alignment.
4.2 System 2: Automated Adaptive-vs-Locked Algorithm Audit Matrix
Objective
Mitigates legal and safety risks associated with artificial intelligence implementations in Software as a Medical Device (SaMD) or Software in a Medical Device (SiMD). It monitors if the firmware utilizes continuous learning protocols that mutate machine learning model parameters after market entry.
Mechanism
Semantic Text Profiling: Audits software specification logs, lifecycle files, and verification documentation for words describing continuous calibration or learning (e.g., adaptive, dynamic weight, real-time reinforcement learning, automated weight calibration, training-on-the-fly).
Validation of Compliance Protocols: Under TFDA rules, any adaptive updates require a predefined "Algorithm Change Protocol (ACP)" or "Post-Market Performance Monitoring and Verification Protocol" (上市後性能監測與驗證機制影本).
Interactive Action: If the system detects adaptive learning patterns but no post-market model maintenance plan, it updates standard clause 8-7, raises a high-risk status indicator, and provides a template for a deficiency prompt:
"經查，本案核心演算法具備動態演進、自我校正之 Adaptive 特徵，惟送審卷宗技術卷二並未檢附符合 TFDA 最新醫療器材人工智慧軟體查驗登記指引要求之『上市後性能監測與驗證機制草案』。請補充說明或聲明鎖定模型（Locked Algorithm）之機制..."
4.3 System 3: Deep Cross-Dataset Data Contamination Detector
Objective
Flags structural validation fraud or mathematical design errors in machine learning evaluation models. It detects when training database entities leak into independent validation or model-testing subsets, which can artificially inflate validation metrics.
Mechanism
Dataset Schema Extraction: Pulls record counts, metadata descriptions, patient indexes, clinical recording names, and patient IDs from training reports and cross-references them against test documents.
Intersection Intersection Matrix Analysis: Automatically flags overlap between supposedly distinct patient sets:
Contamination Threat Index: If identical data segments or patient markers (e.g., patient records Pt-0001 through Pt-0150 in the demo set) appear in both folders, a data leakage alarm is raised.
Interactive Action: Generates a 2D scatterplot in Recharts mapping similarity distributions. It assigns a danger warning to clause 8-7 and outlines a strategy for re-splitting and validating the cohorts to ensure objective performance evaluation.
4.4 System 4 (New): Predictive Regulatory Pathway Classifier & Strategy Advisor
Objective
Provides automated guidance upon initial document upload to determine the exact product classification, product code (CCC Code), and corresponding TFDA review board classification under Taiwanese registration laws.
Mechanism
Classification Mapping Engine: Parses references to device features (e.g., ECG, heart monitoring, catheter, orthopedic plate, surgical laser) and cross-references standard international classification criteria.
CCC-Code Classification Prediction: Predicts classification levels (Class I, II, or III) using keyword patterns and suggests the correct international Harmonized System / CCC Code (e.g., 9018.11.00.00-1 for electrocardiograph devices).
Strategic Intersession Mapping: Determines which specialized TFDA review board should evaluate the file (e.g., 維護心血管系統醫材審查小組 / Cardiovascular Device Committee).
Output Design: Displays a KPI panel within Card 1 of the dashboard, showing the recommended regulatory pathway, testing requirements, and administrative considerations.
4.5 System 5 (New): Legislative Deficiency Draft Synthesis & Statutory Language Alignment Engine
Objective
Translates technical gaps into formal, statutory language. This automated drafting tool formats findings to match official TFDA regulatory communications.
Mechanism
Deficiency Mapping Engine: Bridges identified technical gaps directly with articles of the Taiwan Medical Device Act and associated guidelines (e.g., 醫療器材管理法第33條 (標籤刊登說明書規範) or 第35條 (變更登記限制)).
Deficiency Notification Compilation: Generates formal, traditional Chinese legal drafts:
Example template mapping address issues:
"依據醫療器材管理法第 33 條暨醫療器材許可證核發與登錄及年度申報準則第 15 條第 1 款規範，查驗登記申請書登載之製造廠地址，必須與製售證明及品質管理系統(QSD)登錄核准函之廠址一致。本案申請書地址與QSD核定函格式有間，不符前揭法規規定，應予補正..."
Real-Time Integration: Updates are mirrored in real-time in the terminal output, ensuring any changes to checklist items are instantly reflected in the draft notification document.
4.6 System 6 (New): Pre-clinical Accredited Laboratory Validation & Verification Audit
Objective
Verifies the international credentials and certifications of testing laboratories. If pre-clinical safety testing (e.g., IEC 60601-1 or IEC 60601-1-2) is conducted by unaccredited testing sites, the reports can be outright rejected.
Mechanism
Lab Metatag Scraper: Extracts the names of testing facilities, laboratory certification registrations, ISO standards, and country codes from technical report headers.
Global Accreditation Cross-Checks: Cross-references laboratory IDs against accredited databases (such as Taiwan Accreditation Foundation (TAF), ILAC MRA directory, or A2LA).
Risk Scoring Engine: Assesses the reliability of testing claims based on standard accreditation benchmarks. Evaluates laboratories on a 0 to 100 compliance scale:
Score 100: TAF-certified or verified under the US FDA ASCA program.
Score 0: Lacks verifiable accreditation markers.
Reporting Mechanism: Feeds data directly into Graph 5 (Testing Standards Coverage Area Chart), providing clear warnings about potential test credibility or accreditation issues.
5. Visual Dashboard Specifications (6 Graphs)
To provide clear, visual insights into compliance status and the execution flow of the AI agent, the systems interface includes a Recharts-powered dashboard with six distinct, highly informative graphs.
code
Code
┌─────────────────────────────────────────────────────────────────────────────┐
│                       INTERACTIVE COMPLIANCE VISUAL DASHBOARD               │
├──────────────────────────────────────┬──────────────────────────────────────┤
│  [Graph 1] Sector Compliance Radar   │  [Graph 2] Audit Task Status Pie     │
│  - Captures 8 core chapters balance  │  - Real-time Pass/Fail count tally   │
│  - Radial scoring mapping            │  - Inner-outer responsive layers     │
├──────────────────────────────────────┼──────────────────────────────────────┤
│  [Graph 3] Data Contamination Scatter│  [Graph 4] Address Mismatch bar      │
│  - Overlap check training vs validation│  - Spelling distance comparison     │
│  - Spot outliers & leaks             │  - Displays corporate entity paths   │
├──────────────────────────────────────┼──────────────────────────────────────┤
│  [Graph 5] Standards Coverage Area  │  [Graph 6] Cognitive Run Timeline    │
│  - Safety & Software requirements   │  - Token processing times vs         │
│  - ISO/IEC threshold alignments      │    classification accuracy rates     │
└──────────────────────────────────────┴──────────────────────────────────────┘
5.1 Graph 1: Sector Compliance Radar Chart (8 Chapters Balance)
Objective: Visualizes overall compliance progress across the 8 primary TFDA regulatory chapters.
Component Type: RadarChart inside Recharts, with absolute polar grids.
Data Mapping:
Y-axis / Angle: Chapters (subject: 行政行政、品質管理、技術安規、中文標籤、生物相容、軟體確效).
Area Value: Chapter scores (value: 0% to 100% compliant).
Color Palette: Outer line styled in Indigo (#6366F1) with a transparent blue interior (fillOpacity={0.2}). Allows reviewers to see compliance weaknesses at a glance.
5.2 Graph 2: Audit Task Status Doughnut Chart (Status Distribution)
Objective: Displays the real-time breakdown of current progress across the 40+ statutory checklist clauses.
Component Type: Double-radius PieChart with active center-key text showing total clauses.
Data Mapping:
Categories: Pass (#10B981 Emerald), Fail (#EF4444 Rose), N/A (#6B7280 Slate Gray), and Pending (#F59E0B Amber).
Design Pattern: Responsive size, hover highlights, and customized tooltip elements designed for touch targets.
5.3 Graph 3: Training-vs-Test Similarity Scatterplot (Leakage Identification)
Objective: Visualizes potential data leakage or cross-contamination in ML training models.
Component Type: ScatterChart with overlapping datasets, grid styling, and custom label vectors.
Data Mapping:
X-axis: Training similarity index (%).
Y-axis: Testing similarity index (%).
Scatter Points: Overlapping Patient ECG profiles. Red markers reflect data leakage, while blue markers designate independent verification datasets.
5.4 Graph 4: Entity Address Mismatch Distance Bar Chart (Audit Matches)
Objective: Compares spelling consistency across the various corporate directory address lines submitted.
Component Type: Horizontal double-axis BarChart for rapid vertical reading.
Data Mapping:
Y-axis: Submission Document Sources (CFSM, Application, LoA, QSD).
X-axis: Text similarity index (0-100%).
Design Pattern: Dynamic colors. Scores below 95% trigger immediate red alerts, while consistent matches remain a neutral slate gray.
5.5 Graph 5: Testing Standards Coverage Area Chart (Required vs Realized)
Objective: Tracks overall compliance across required international standards.
Component Type: Linear gradient AreaChart with shadow gradients.
Data Mapping:
X-axis: Reference Standards (IEC 60601-1, IEC 62304, ISO 14971, ISO 10993).
Y-axis: Verification Score (0-100%).
Design Pattern: A filled grid pattern showing standard alignment profiles, helping identify uncompleted or missing pre-clinical laboratory test suites.
5.6 Graph 6: Cognitive Run Timeline & Confidence Plot (System Efficiency)
Objective: Deep research model performance diagnostic graph designed to visualize resource consumption versus classification confidence values during multi-agent turns.
Component Type: Mixed ComposedChart combining Line grids and Area points.
Data Mapping:
X-axis: Parsing Turn Index (
).
Y1-axis: Total tokens processed.
Y2-axis: Classification accuracy rating & model confidence levels (0% to 100%).
Design Pattern: Displays the technical performance of the underlying processing models during compliance sweeps.
6. Living Log & Cognitive Console Design
To improve user confidence in the AI results, the system includes a terminal-style cognitive log viewer that reveals real-time assessment dynamics as they occur behind the scenes.
code
Code
================================================================================
 [AGENT REACTION WORKSPACE CONSOLE - V2026.06] STATUS: ACTIVE
================================================================================
 > [SYSTEM] Ingesting traditional 510(k) heart cardiac monitoring software...
 > [PROCESS] Ingesting Address strings...
 > [WARN] Address mismatch detected below 95% similarity ceiling:
          CFSM: "100 Innovation Way, Suite 401, Boston, MA"
          Application: "100 Innovation Way, 4F Rm 401, Boston, MA"
 > [WARN] Computing string distance... Levenshtein Score: 78.4%
 > [ALERT] Algorithm architecture pattern scanning... Detected "continuous weight updates".
 > [DANGER] REGULATORY GAP: No "Post-Market Verification Protocol" provided for Adaptive SaMD.
 > [PROCESS] Cross-checking dataset overlaps...
 > [DANGER] 150 overlapping patients detected between Train & Test pools! (Leakage flag).
 > [SUCCESS] Technical compliance index computed: 51.2% adequacy rate.
================================================================================
6.1 Subsystem Layout and Style Guidelines
Container Styling: Framed in deep obsidian slate (bg-slate-950), absolute flat dark border (border-slate-800), and a terminal monospace font stack (font-mono).
Visual Status Anchors: Red-Yellow-Green terminal lights indicating real-time status.
Action Logs & Dynamic Traces: Custom, color-coded logging blocks that differentiate levels of threat or progress at a glance:
[SYSTEM]: Blue indicators displaying initialization vectors.
[PROCESS]: Indigo markers denoting active evaluation sub-segments.
[WARN]: Amber notifications highlighting minor compliance deviations.
[DANGER]/[ALERT]: Glowing crimson warnings representing serious non-compliance or data contamination.
[SUCCESS]: Bright emerald indicators validating completed certifications.
7. Interactive Review Worktable & State Management
The core of the compliance workflow is the split-screen, interactive accordion checklist layout. The system uses a two-column functional layout to optimize screen space and focus.
code
Code
┌─────────────────────────────────────────────────────────────────────────────┐
│                                SYSTEM SPLIT SCREEN                          │
├──────────────────────────────────────────┬──────────────────────────────────┤
│           COLUMN 1 (Checklist Items)     │    COLUMN 2 (Deficiency Output)  │
│                                          │                                  │
│  [Chapter Accordion] (e.g., 5、QMS 審查) │  ┌────────────────────────────┐  │
│  ┌────────────────────────────────────┐  │  │ Real-time deficiency letter│  │
│  │ 5-1: QMS Certificate 3-year term   │  │  │ text output box.           │  │
│  │ ├─ Status: [PASS] [FAIL] [N/A]     │  │  │                            │  │
│  │ ├─ Rule: "Must be valid within..." │  │  │ - Formatted with legal text│  │
│  │ └─ Notes: User annotations go here  │  │  │ - Links to Taiwan Med Act  │  │
│  └────────────────────────────────────┘  │  └────────────────────────────┘  │
│  [Chapter Accordion] (e.g., 8、技術審查) │  ┌────────────────────────────┐  │
│  ┌────────────────────────────────────┐  │  │ Action Buttons:            │  │
│  │ 8-7: AI/ML Train/Test Data Split   │  │  │ [Copy] [G-Doc] [Markdown]  │  │
│  │ ├─ Status: [FAIL]                  │  │  └────────────────────────────┘  │
│  │ ├─ AI Suggest: "Pt dataset leakage"│  │                                  │
│  │ └─ [Adopt AI Action Suggestion]    │  │                                  │
└──────────────────────────────────────────┴──────────────────────────────────┘
7.1 Accordion Component Specifications
Section Headers: Responsive click paths that dynamically toggle active sections. Displays the total number of items, along with cumulative Pass, Fail, or N/A statuses for that chapter.
Clause Item Viewports:
Status Selectors: Multi-state switches that let reviewers change status between Pass, Fail, or N/A with instant state updates.
Statutory Rule Block: Displays the official legal requirement, decorated with a neutral slate-gray border.
Reviewer Notes Area: Full-width editing textareas. State changes are serialized as they are typed, triggering updates both in local storage (localStorage) and in the generated deficiency report draft.
AI Strategic Advisor Widgets: Interactive alert frames that appear if an item is marked "Fail." These widgets show AI recommendations, calculate similarity margins, and feature an "Adopt and write to comments" (採納建議並寫入工作台) button, allowing reviewers to instantly populate the notes field with professional, formatted regulatory phrasing.
8. Export Control Pipeline Specifications
To support varied writing workflows, the output can be exported as a highly-structured review report via three main format options:
code
Code
┌──────────────────────────┐
                          │    State Report Draft    │
                          └────────────┬─────────────┘
                                       │
                ┌──────────────────────┼──────────────────────┐
                ▼                      ▼                      ▼
     ┌────────────────────┐ ┌────────────────────┐ ┌────────────────────┐
     │ 1. Markdown (.md)  │ │ 2. G-Doc Scraps    │ │ 3. Print HTML      │
     │  - Strict structure│ │  - Clear spacing   │ │  - Self-contained  │
     │  - Code block grids│ │  - Simplified copy │ │  - Embedded styles │
     └────────────────────┘ └────────────────────┘ └────────────────────┘
8.1 1. Markdown Technical Dossier (.md)
Architecture: Standard UTF-8 document containing structured headers, KPI tables, and compliance scoreboards.
Layout Structure:
Executive overview detailing the target device and testing parameters.
Real-time metrics breakdown, including Pass, Fail, and Pending item tallies.
Consolidated technical deficiency notification letter, structured for legal communication.
Detailed logs and assessment notes for all 40+ statutory checklist clauses.
8.2 2. G-Doc Scratchpad Copy Mode (.txt)
Architecture: A copying-optimized text structure formatted to simplify pasting into document editors, such as Microsoft Word or Google Docs.
Layout Structure:
Removes markdown layout tokens and other code blocks that degrade pasting.
Formatted with double newline spaces to ensure clean text blocks and preserve typographical hierarchies when pasted.
Pre-formatted as a regulatory draft, including official TFDA formatting placeholders and article alignments.
8.3 3. Printable Stand-alone Technical Report (.html)
Architecture: A single-file, self-contained HTML page containing embedded Tailwind CSS rendering styles.
Layout Structure:
Fully styled vector headers, dashboard progress components, and compliance status indicators.
Grouped accordions displaying the status of audited chapters.
Layout designed to ensure compatibility with both internal archiving processes and local printing pipelines.
9. Comprehensive Regulatory Requirements Dataset Blueprint
The application database maps the official regulatory checklists from the卫福部 TFDA "醫療器材許可證核發與登錄及年度申報準則" into an interactive dataset of eight primary chapters:
Chapter 1: Administrative Form Audit (申請書資訊核對)
1-1 (中英文繕打): Verifies the medical device application form is thoroughly completed in both Traditional Chinese and English, including verified seals from both the company and the responsible officer.
1-2 (型號規格吻合): Validates that the Chinese name, English brand name, model numbers, and accessory codes match exactly across the application, CFSM, and LoA documents.
1-3 (品名核定原則): Confirms compliance with TFDA naming guidelines (prohibiting unauthorized trademark name use or misleading health and efficacy claims).
1-4 (加冠商標協議書): Verifies that appropriate licensing context or trademark certificates are provided for any branded identifiers on the device casing.
1-5 (申請醫材商地址): Automatically matches the registered address of the applicant against their official medical device medical merchant registration.
1-6 (製造廠址檢對): Compares the manufacturer's name and physical address across standard documents, checking for spelling address discrepancies.
1-7 (委託製造核定): Inspects manufacturing contract relationships, verifying the correct "Manufactured by [Entity A] for [Entity B]" relational structure.
1-8 (型號系列原則): Confirms that different model types belong to the same technical series, flagging variations that require separate registration applications.
Chapter 2: Mainland China Medical Imports Check (陸輸醫療器材與資格)
2-1 (CCC Code 登錄): Confirms the correct categorization codes are listed under TFDA trade classifications.
2-2 (國貿局輸入許可): Checks for required import permits if importing regulated devices or materials under restricted trade lists.
2-3 (輸入販賣許可影本): Verifies that the applicant’s medical merchant license includes import clearance permissions.
2-4 (製造業許可執照): For domestic devices, verifies that manufacturer permits match the production address.
2-5 (委託製造授權登錄影本): Confirms active contractual records have been logged in the national database.
Chapter 3: Certificate of Free Sale & Manufacture (製售證明 - CFSM)
3-1 (證書正本檢核): Verifies that the physical Certificate of Free Sale and Manufacture (CFSM) is valid and registered with the appropriate reference number.
3-2 (主管機關出具權責): Verifies the certificate was issued by the highest health authority in the manufacturer's home branch (e.g., US FDA).
3-3 (代表處驗證章): Confirms proper diplomatic authentication by ROC foreign missions (unless exempted under official bilateral agreements).
3-4 (中英文譯本公證): Confirms that non-English certificates feature validated Traditional Chinese translations.
3-5 (本國銷售合法宣告): Verifies the document includes explicit confirmation that the device is sold legally in its home market.
3-2 (有效期限二年): Ensures the CFSM was issued within the last two years, flagging expired documents for corrective action.
3-7 (製造商名稱地址核對): Cross-compares manufacturer addresses between the CFSM and the application form to ensure consistency.
3-8 (配件規格清冊): Verifies that all target device model variations and sub-components are explicitly covered by the CFSM.
Chapter 4: Foreign Manufacturer Letter of Authorization (國外原廠授權登記書 - LoA)
4-1 (授權書正本效力): Validates that the original physical Letter of Authorization (LoA) is current and active, checking against official records.
4-2 (合法授權關係鏈): Traces relationships back to the ultimate holding company if authorization is routed through intermediary organizations.
4-3 (出具日起一年效期): Ensures that the authorization letter is within its one-year validity window.
4-4 (原廠名稱地址一致): Verifies the foreign manufacturer address exactly matches corresponding directories.
4-5 (配合醫材管理承諾): Confirms the LoA includes explicit agreements to comply with domestic medical device regulations and reporting criteria.
4-6 (中文譯本核退): Verifies non-English authorizations include certified Chinese translations.
4-7 (明列型號規格範疇): Checks that the document specifies individual device model types or designates all company products.
4-8 (代理商名稱地址勾稽): Verifies that the name and physical address of the domestic agent align with local business registros.
4-9 (委託代工授權): When using contract manufacturing models, confirms the LoA is issued by the brand owner and includes contract manufacturer details.
Chapter 5: Quality Management System Compliance (品質管理系統證明 - QSD / GMP)
5-1 (QSD 核發許可函): Verifies the applicant holds a valid QMS/QSD registration letter, confirming current status.
5-2 (醫材商名稱相符): Confirms that the holder listed on the QMS certificate is identical to the main applicant company.
5-3 (製造廠名地址勾稽): Cross-compares names and addresses on the QMS certificate with the application form.
5-4 (登錄效期三年): Verifies that the current QMS/QSD registration has not expired and is within its standard three-year term.
5-5 (登錄登錄品項代碼): Checks that the registration is coded for the specific medical device category under review.
5-6 (QSD 覆蓋品項確認): For complex devices, confirms that the QMS registration covers the specific manufacturing processes used (such as sterilization or final release).
5-7 (受託滅菌QSD): If sterilization is outsourced, checks that the contract site holds appropriate QMS/QSD certifications.
5-8 (包裝貼標受託廠QSD): If packaging or labeling is handled by contractors, confirms they are properly certified under QMS regulations.
Chapter 6: Manufacturing Contracts & Agreements (委託製造合約與證明)
6-1 (製程分工說明文件): Confirms the presence of technical process charts mapping the processing responsibility of each participating sub-facility.
6-2 (國內全部製程核准函): Considers whether domestic outsourced factories hold active permits for the complete manufacturing run.
6-3 (國內委託製造契約): Inspects manufacturing agreements, verifying they contain explicit liability clauses and quality controls.
6-4 (國外製造廠委託合約): Verifies that international production contracts clearly establish responsibility and are validated by foreign authorities.
Chapter 7: Packaging, Labeling & Instructions for Use (標籤及說明書)
7-1 (中文標籤說明書擬稿): Verifies the layout draft includes required items (lot number, manufacture date, expiration date, name, and address of chemical merchants).
7-2 (原廠外盒包裝實貼): Checks for the inclusion of real photo packaging artwork mapping the device origin and identification codes.
7-3 (原廠說明書及型錄): Verifies the inclusion of original manufacturer catalogs and operational handbooks.
7-4 (產品彩色照片與實配): Confirms presence of complete multi-angle color photographs of the physical housing and accessories.
7-5 (中譯仿單擬稿): Verifies that Chinese translations of manuals accurately reflect the original manufacturer specifications.
7-6 (委託製造登載文字): Verifies that final packaging drafts correctly state "Manufactured by [Entity A] for [Entity B]."
Chapter 8: Technical Review & Testing Verification (技術審查與安全確效)
8-1 (電氣與相容性測試): Verifies inclusion of comprehensive IEC 60601-1 and IEC 60601-1-2 EMC test reports from accredited sites.
8-2 (生物相容性報告): Confirms clinical validation reports mapping sensitization and cytotoxicity for any patient-contact components (ISO 10993).
8-3 (臨床前功能性測試): Audits specific engineering tests demonstrating the technical performance of the device against specifications.
8-4 (特定設備安規確效): Checks for the inclusion of specialized technical reports (such as laser calibration, acoustic index levels, or radiation safety measures).
8-5 (滅菌確效安定性報告): Inspects technical reports mapping sterilization validation (ethylene oxide residue tests) and shelf-life stability tests.
8-6 (軟體確效生命週期): Inspects IEC 62304 verification records showing proper software life cycle controls and security sweeps.
8-7 (人工智慧臨床驗證): Checks data separation structures for AI algorithms, ensuring no overlaps exist between training and testing sets.
8-8 (清潔與重複用重製程序): For reusable equipment, validates technical instructions describing validated sterilization steps.
10. Under-The-Hood Architecture Details
To ensure responsive performance without typical framework flickering issues, the system includes custom state managers, drag-and-drop file readers, and optimized layout code.
10.1 App Ingestion & State Container Controller Model
The React application leverages dynamic, native states to prevent lag or latency when processing large technical files:
code
TypeScript
// Core State Blueprint within App.tsx
const [modelName, setModelName] = useState<string>('gemini-3.5-flash');
const [materialsText, setMaterialsText] = useState<string>('');
const [skillText, setSkillText] = useState<string>('');
const [isAnalyzing, setIsAnalyzing] = useState<boolean>(false);
const [checklist, setChecklist] = useState<ReviewSection[]>(INITIAL_CHECKLIST_DATA);
const [logs, setLogs] = useState<string[]>([]);
Storage Sync: Checklist evaluations, reviewer notes, and similarity scores are serialized to localStorage continuously. This ensures that progress is preserved if the browser is closed or refreshed during an audit.
Vite Integration: The server handles asset generation and hot replacement dynamically, providing a seamless live-preview workflow in the AI Studio environment.
10.2 API Proxy Server Route (/api/review)
To protect sensitive API tokens, the application routes AI model communication through an Express proxy. The server receives the submission payload, executes initial diagnostic checks, and connects to the GenAI SDK. If the API key is not present, the route falls back to standard heuristic audits.
code
TypeScript
import { GoogleGenAI } from "@google/genai";

// Fast check for active API credentials
function isApiKeyValid() {
  const k = process.env.GEMINI_API_KEY;
  return !!k && k !== "MY_GEMINI_API_KEY" && k.trim() !== "";
}

app.post("/api/review", async (req, res) => {
  const { materials, skillMd, modelName } = req.body;
  const logs: string[] = [];
  
  logs.push(`[INFO] Initializing review processing stream...`);
  
  if (isApiKeyValid()) {
    try {
      const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });
      const prompt = `...Detailed instructions describing regulatory checklist parsing ...`;
      
      const response = await ai.models.generateContent({
        model: modelName,
        contents: prompt,
        config: {
          responseMimeType: "application/json",
          temperature: 0.1,
          systemInstruction: "You are a professional Taiwanese TFDA medical device registration reviewer."
        }
      });
      
      const parsedReport = JSON.parse(response.text);
      return res.json({ success: true, logs, result: parsedReport });
    } catch (e: any) {
      logs.push(`[WARN] Model processing error: ${e.message}. Activating local fallback parsing...`);
    }
  } else {
    logs.push(`[INFO] No credential key detected. Accessing local regulatory heuristics...`);
  }
  
  // Heuristic-driven analysis fallback (simulated metrics compilation)
  const localAnalysis = compileLocalHeuristics(materials, skillMd);
  res.json({ success: true, logs, result: localAnalysis });
});
10.3 Compilation and Production Bundling Configuration
The package.json and build scripts compile the client application into unified statutory assets while bundling the backend server for minimal overhead:
code
JSON
"scripts": {
  "dev": "tsx server.ts",
  "build": "vite build && esbuild server.ts --bundle --platform=node --format=cjs --packages=external --sourcemap --outfile=dist/server.cjs",
  "start": "node dist/server.cjs"
}
By deploying via esbuild with --format=cjs, the backend server is compiled to standard CommonJS format (dist/server.cjs). This bypasses ES Module runtime path alignment issues, resulting in clean, predictable execution across all target development and runtime environments.
11. Architectural Follow-Up & Verification Questions
To align implementation strategies with organizational goals, we define twenty architectural follow-up questions to guide the development and deployment phases:
TFDA Code Adaptations: Does the organization wish to include specific compliance rules for In Vitro Diagnostics (IVD) or Class III high-risk mechanical implants within the default skill.md mapping?
OCR Integration Capabilities: Should we integrate a server-side PDF parser with Optical Character Recognition (OCR) to extract text from scanned, stamped manufacturing licenses?
Google Doc Integration Method: Should we connect the document export feature directly to the Google Drive API using OAuth, or is the current local copying scratchpad sufficient?
Database Strategy: For persistent, collaborative reviews involving multiple users, should we transition from local browser storage to a server-side Firebase Firestore database?
Data Privacy Constraints: Should we implement client-side data anonymization to strip Patient Name records, National ID numbers, or proprietary source code elements before sending documents to the GenAI API?
Local LLM Hosting Options: If highly strict data privacy is required, should we support routing API requests to clean local enterprise models (such as private Vertex AI endpoints)?
Address Normalization Variations: Are there additional local address formats or regional abbreviations that we should add to our text normalization rules to prevent false-alarm discrepancies?
Clinical Cohort Safety Threshold: For the data leakage metric, what is the exact percentage overlap threshold that should trigger a critical regulatory failure versus a simple compliance warning?
Laboratory Registry Sync: How frequently should the pre-clinical laboratory database be updated with new accreditation details from TAF and ILAC directories?
Custom Checklist Modifications: Do regulatory managers need an admin interface to dynamically add or edit checklist questions without modifying the raw JSON databases?
Collaborative Flow Management: To support team workflows, should we implement features to lock clinical chapters or assign specific sections to distinct audit team members?
Version Control Integration: Should the system track historical iterations of technical dossiers, allowing reviewers to compare changes across different submission versions?
Vite Routing Selection: Do we need to implement multi-page React routing (such as React Router), or is our single-page layout better suited for preserving the state of complex review sessions?
Custom PDF Export Requirements: For printable review documents, should we use server-side generation engines (like headless Puppeteer) to produce formal, PDF-compliant layout forms?
HMR Error Troubleshooting: Since Hot Module Replacement (HMR) is disabled in AI Studio to prevent flickering during edits, should we provide specific configuration overrides for teams working in local development environments?
AI Confidence Level Scoring: How should we calculate the overall confidence level shown in the timeline chart when matching multiple complex regulatory questions?
Workspace API Key Management: Should the workspace include a secure, temporary interface for entering API credentials, or should it rely entirely on server-side environment variables?
Expanded Industry Alignments: Are there other international regulatory standards (such as US FDA 510(k) or EU MDR) that we should integrate into our standard compliance evaluation matrices?
Response Latency Optimizations: Should we implement stream-rendering techniques for LLM responses to show evaluations and notes on individual items as they are processed?
Post-Market Protocol Assistance: Should we include interactive wizards that help product manufacturers write compliant Post-Market Performance Monitoring and Verification Protocols?
