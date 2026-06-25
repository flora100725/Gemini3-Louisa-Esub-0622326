# ARCHITECTURAL DESIGN & TECHNICAL SPECIFICATION: MULTI-REGULATORY MEDICAL DEVICE KNOWLEDGE ENGINE (AGENTIC SYSTEM)

---

## 1. SYSTEM OVERVIEW & WORKFLOW ARCHITECTURE

The Multi-Regulatory Medical Device Knowledge Engine is an enterprise-grade, agentic AI platform engineered to harmonize global medical device nomenclatures with localized unique device identification registries. The architecture specifically integrates two distinct master data domains:

1. **MEDIVID Nomenclature Dataset:** Formatted as structured taxonomies maps device descriptions to European Medical Device Nomenclature (EMDN) and Global Medical Device Nomenclature (GMDN) standard codes, definitions, and contextual schemas.
2. **Taiwan Unique Device Identifier Database (TUDID):** Governed by the Taiwan Food and Drug Administration (TFDA), tracking Device Identifiers (DIs), license status, registration types, and risk categorizations.

The operational objective of this system is to bridge the gap between global standardization frameworks and localized regulatory oversight through an autonomous semantic layer powered by advanced LLM orchestration.

### End-to-End System Architecture

```
                                  +-----------------------+
                                  |   Streamlit Interface |
                                  +-----------------------+
                                   /                     \
       (Data Ingestion / File Upload)                   (Natural Language Query)
                 v                                                 v
  +-------------------------------+                 +------------------------------+
  |  Ingestion & Validation Engine|                 |   Agentic Text-to-SQL Layer  |
  +-------------------------------+                 +------------------------------+
    | CSV/JSON       | Default Data                   | Dynamic Schema Resolution   |
    v                v                                | Safety/Sanitization Checks  |
+---------------------------------+                   v                              |
|   DuckDB Local OLAP Storage     |<------------------ Executing Validated SQL -------+
+---------------------------------+                               |
                 |                                                v
                 | Extract Subset Data                   +------------------+
                 +-------------------------------------->| Analytics Engine |
                                                         +------------------+
                                                                  |
                                                                  v
                                                         +------------------+
                                                         |  Matplotlib/Seaborn |
                                                         +------------------+

```

### High-Level Blueprint and Module Breakdown

The system is decoupled into four primary architectural tiers:

* **Data Ingestion and Schema Enforcement Module:** Responsible for handling external file inputs (CSV, JSON) and establishing fallback connections to the default built-in dataset structures. It manages type coercion, handles null values, and validates regulatory schemas using an ultra-fast in-memory columnar database.
* **Agentic Text-to-SQL Translation Layer:** Converts unstructured, multi-lingual linguistic parameters into structured analytical SQL dialect commands. It routes tasks dynamically depending on the selected model configuration, utilizing `gemini-3.1-flash-lite` as the default engine.
* **Decoupled Rendering and Export Subsystem:** Isolates data retrieval from representation. This tier constructs high-throughput, dynamic HTML tables and maps downstream tabular streams into downloadable markdown, CSV, structured JSON, or standalone web files.
* **Deterministic Python Visualization Worker:** Receives structural data slices, safely reads data configurations, and maps information to a collection of exactly 5 specific charts. This module guarantees consistent rendering patterns without allowing arbitrary, unverified code executions.

---

## 2. DATASET SUMMARY & SEMANTIC ANALYSIS

### Comprehensive Analysis of the Supplied Registries

#### MEDIVID Nomenclature Master File

The MEDIVID dataset operates as a standardized terminology reference mapping cross-border medical device designations to formalized structural nomenclature frameworks.

```
+----------------------------------------------------------------------------------------------------------------+
|                                           MEDIVID DATASET SCHEMA                                               |
+---------------------+-------------------+---------------------+--------------------+---------------------------+
| Field Name          | Data Type         | Constraint          | Primary Mapping    | Semantic Purpose          |
+---------------------+-------------------+---------------------+--------------------+---------------------------+
| Device Name         | VARCHAR           | NOT NULL, INDEXED   | Literal Descriptor | Standard market/clinical  |
| Nomenclature (EMDN) | VARCHAR (Code)    | NULLABLE            | European Registry  | Alpha-numeric categorization|
| Nomenclature Term   | VARCHAR           | NULLABLE            | EMDN Classifier    | Category description      |
| Nomenclature (GMDN) | INTEGER / VARCHAR | NOT NULL, UNIQUE KEY| Global Registry    | International numeric ID  |
| GMDN Definition     | TEXT              | NOT NULL            | Scope Boundaries   | Functional criteria       |
+---------------------+-------------------+---------------------+--------------------+---------------------------+

```

##### Data Profile & Structural Topography

The dataset provides direct semantic bridges across differing frameworks. Analysis reveals precise text descriptors mapped explicitly to structural categories. For example, manual soft-tissue manipulation tweezers are mapped directly to GMDN classification code **62466** ("Surgical soft-tissue manipulation forceps, tweezers-like, reusable"), paired alongside strict EMDN code alignments like **L1106** ("NEUROSURGERY FORCEPS, REUSABLE"). Concurrently, standard consumer and clinical items like absorbent applicators map to GMDN code **33722** under EMDN category **M0499** ("SPECIAL DRESSINGS - OTHER").

##### Semantic Distribution and Structural Characteristics

The dataset is dense, featuring granular technical prose inside its definitions. Its structure shows zero truncation but introduces severe semantic overlap where various distinct contextual devices share single overarching GMDN classifications due to shared core clinical modalities.

```
       [MEDIVID Vocabulary Density Map]
High   +---------------------------------------+
       | GMDN Definitions (Long Form Text)    |
       | - High technical granularity          |
       | - Dense functional boundaries         |
       |                                       |
       | Device Names (Short Form Text)        |
       | - High variation, literal descriptors |
Low    +---------------------------------------+

```

#### Taiwan UDID (TUDID) Registry File

The TUDID dataset represents a localized legal registry tracking medical device imports and domestic products authorized by the TFDA. It includes detailed multi-lingual tracking metrics, product classifications, and global supply chain indicators.

```
+----------------------------------------------------------------------------------------------------------------+
|                                            TUDID DATASET SCHEMA                                                |
+---------------------+-------------------+---------------------+--------------------+---------------------------+
| Column Name (Raw)   | DB Target Field   | Target Type         | Constraint         | Regulatory Purpose        |
+---------------------+-------------------+---------------------+--------------------+---------------------------+
| 許可證字號（類型）   | license_no        | VARCHAR (PK Component) NOT NULL           | License Serial ID         |
| UDI發碼機構          | udi_agency        | VARCHAR             | NOT NULL (e.g. GS1)| Standard Issuing Body     |
| 基本DI              | primary_di        | VARCHAR (Indexed)   | NOT NULL           | Global Trade Item Basis   |
| 型號                | model_no          | VARCHAR             | NULLABLE           | Manufacturer Part Number  |
| 中文品名            | Chinese_name      | VARCHAR             | NOT NULL           | Legal Chinese Designation |
| 註銷狀態            | cancellation_status VARCHAR           | NULLABLE           | Active/Revoked Marker     |
| 醫療器材級數        | risk_class        | INTEGER/SMALLINT    | Range [1, 2, 3]    | Statutory Risk Grade      |
| 申請商名稱          | applicant_name    | VARCHAR             | NOT NULL           | Legal Sponsoring Entity   |
| 醫器次類別一         | sub_category_1    | VARCHAR             | NOT NULL           | Primary Medical Subclass  |
+---------------------+-------------------+---------------------+--------------------+---------------------------+

```

##### Deep Profile and Field Interrelationships

The structural relationships in this dataset are tightly integrated. The data links high-risk life-support hardware, like implantable cardioverter-defibrillators (ICDs) from global manufacturers, to localized regulatory attributes. For instance:

* **License Number:** `衛署醫器輸字第024889號` (indicating an imported Class III medical device).
* **Primary DI Structure:** Utilizes standard GS1 global formats (`00643169007147`).
* **Risk Class:** Assigned to strict Class 3 boundaries.
* **Medical Subcategory Type:** Categorized under specialized functional classifications like `E.3610 植入式心律器之脈搏產生器` (Implantable Pacemaker Pulse Generator).

Lower-risk diagnostic modalities, such as physiological patient monitors, are tracked using localized codes like `衛部醫器陸輸字第000858號`, mapping directly to risk Class 2 under the `E.2340 心電圖描記器` subcategory.

### Cross-Dataset Syntactic and Semantic Gaps

Connecting MEDIVID and TUDID presents clear challenges due to differences in schema design, nomenclature frameworks, and multi-lingual naming conventions:

```
+----------------------------------------------------------------------------------------------------------------+
|                                    STRUCTURAL CONVERSION CHALLENGES                                            |
+------------------+------------------------------+------------------------------+-------------------------------+
| Attribute        | MEDIVID Dataset              | TUDID Dataset                | Resolution Strategy           |
+------------------+------------------------------+------------------------------+-------------------------------+
| Language Base    | Monolingual English          | Bilingual English/Traditional Chinese | Dual-Embeddings & Synonym Mapping|
| Structural Identifiers | EMDN Codes, GMDN Codes  | Primary DIs (GS1), TFDA Licenses | N-to-N Mapping via Description|
| Granularity Level | Discrete Functional Concept  | Specific Manufacturer SKUs   | Fuzzy String Distance Scoring |
+------------------+------------------------------+------------------------------+-------------------------------+

```

The system overcomes these limitations by implementing cross-attribute matching joins. These joins combine English diagnostic descriptions from MEDIVID with bilingual product entries in TUDID by mapping standardized classification roots to clinical function subcategories.

---

## 3. AGENTIC TEXT-TO-SQL INTERFACE & SCHEMA RESOLUTION

To process natural language requests without risking SQL injection or performance bottlenecks, the agent utilizes a decoupled text-to-SQL mapping framework.

### Conceptual Data Modeling (OLAP Representation)

Upon loading, the pipeline processes the files into two high-performance relational tables managed within an embedded DuckDB instance.

```sql
-- Target Schema definition within DuckDB
CREATE TABLE medivid_nomenclature (
    device_name VARCHAR,
    emdn_code VARCHAR,
    emdn_term VARCHAR,
    gmdn_code VARCHAR,
    gmdn_definition TEXT
);

CREATE TABLE tudid_registry (
    license_no VARCHAR,
    udi_agency VARCHAR,
    primary_di VARCHAR,
    model_no VARCHAR,
    chinese_name VARCHAR,
    cancellation_status VARCHAR,
    risk_class INTEGER,
    applicant_name VARCHAR,
    sub_category_1 VARCHAR
);

```

### Agent Logic and State-Machine Execution Flows

The query planning engine acts as a multi-stage validation pipeline that processes raw user requests into sanitized execution routines:

```
[User Natural Language]
          |
          v
[1. Prompt Sanitization & Structural Mapping]
   - Parse filter criteria (e.g., risk_class = 3)
   - Extract raw text search vectors
          |
          v
[2. LLM Intent & Schema Translation Layer]
   - Reference internal schema map
   - Generate standards-compliant SQL
          |
          v
[3. Multi-Layer SQL Guardrail & AST Validation]
   - Block DDL/DML commands (DROP, INSERT, etc.)
   - Enforce system-level LIMIT boundaries
          |
          v
[4. Query Execution Engine (DuckDB In-Memory)]
          |
          v
[5. Decoupled Presentation Layer Rendering]

```

#### Step 1: Prompt Sanitization and Structural Mapping

The user defines an analytical target through natural language inputs, accompanied by explicit filtering constraints (e.g., specific risk classes or regulatory statuses) applied via the UI panel.

#### Step 2: LLM Intent Translation Layer

The system constructs a secure context wrapper that provides the selected LLM (defaulting to `gemini-3.1-flash-lite`, with optional overrides for larger frontier models) with the core table schemas and a clear set of translation instructions.

```
You are an isolated, read-only SQL generation agent. Translate user input into standard DuckDB SQL.
Only query the tables: medivid_nomenclature, tudid_registry.
Do not generate markdown formatting or explain your output. Output the raw SQL string directly.

```

#### Step 3: Multi-Layer SQL Guardrail and AST Validation

Before the generated query is sent to the database engine, a validation step reviews the syntax tree to ensure safety and stability:

* **Whitelisting Execution Commands:** Only explicit, standard selection phrases (`SELECT`) are permitted. Structural adjustment operations like `DROP`, `ALTER`, `DELETE`, or `INSERT` are blocked immediately.
* **System-Level Pagination Safety:** The system inspects the query and enforces standard output size limits (e.g., capping responses at `LIMIT 1000`) if no explicit threshold is provided.

#### Step 4: Query Execution Engine

The verified SQL query runs against the local in-memory database, returning a structured data frame directly to the UI layer.

#### Step 5: Presentation Layer Rendering

The raw data frames are passed to a rendering engine that formats the information for display and creates downloadable files.

---

## 4. DECOUPLED RENDERING, EXPORT & DETERMINISTIC VISUALIZATION PIPELINE

### UI Grid Presentation Layer

The presentation engine converts the raw dataset arrays into standard responsive HTML tables, styled cleanly via CSS utilities. It supports dynamic sorting and pagination without needing to request fresh data pulls from the underlying database.

### Multi-Format Serialization Architecture

The export interface converts the active data frames into multiple target formats through specific processing routines:

* **Markdown Structure:** Maps the tabular rows into text representations using standard markdown syntax.
* **JSON Serialization:** Converts raw records into standard JSON arrays using structured key-value pairs.
* **CSV Generation:** Outputs text strings containing comma-separated row values with appropriate text wrapping.

### Deterministic Python-Driven Visualization Processing

To prevent security issues associated with running unverified runtime logic, the visualization pipeline avoids executing raw, dynamically generated code strings. Instead, it relies on a deterministic analysis script that accepts structured data arrays and configuration parameters to render a suite of exactly **five core analytical charts**:

```python
# System visualization orchestration core
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np

def generate_analytical_suite(df_input: pd.DataFrame) -> list:
    """
    Accepts arbitrary filtered dataframe outputs and generates 5 deterministic charts
    returning raw HTML container fragments or base64 data channels.
    """
    plot_outputs = []
    sns.set_theme(style="whitegrid")
    
    # Chart 1: Risk Categorization Spectrum (Bar Plot)
    fig, ax = plt.subplots(figsize=(8, 4))
    if 'risk_class' in df_input.columns:
        df_input['risk_class'].value_counts().sort_index().plot(kind='bar', color='#1f77b4', ax=ax)
        ax.set_title("Distribution of Medical Device Risk Classifications")
        ax.set_xlabel("Risk Class")
        ax.set_ylabel("Device Registry Volume")
    plot_outputs.append(fig)
    
    # Chart 2: Top Global Issuing Authorities & Applicants (Horizontal Categorization Bar)
    fig, ax = plt.subplots(figsize=(8, 4))
    target_col = 'applicant_name' if 'applicant_name' in df_input.columns else 'udi_agency'
    if target_col in df_input.columns:
        df_input[target_col].value_counts().head(10).plot(kind='barh', color='#2ca02c', ax=ax)
        ax.set_title(f"Top 10 Entities by Device Volume Presence")
    plot_outputs.append(fig)
    
    # Chart 3: Composition Breakdown of Device Registries (Pie Diagram)
    fig, ax = plt.subplots(figsize=(6, 6))
    if 'cancellation_status' in df_input.columns:
        status_data = df_input['cancellation_status'].fillna('Active').value_counts()
        ax.pie(status_data, labels=status_data.index, autopct='%1.1f%%', colors=['#4b0082', '#ff7f0e'])
        ax.set_title("Device Lifecycle Status Ratios")
    plot_outputs.append(fig)
    
    # Chart 4: Categorical Semantic Cluster Word Length Distribution (Histogram/KDE)
    fig, ax = plt.subplots(figsize=(8, 4))
    if 'gmdn_definition' in df_input.columns:
        lengths = df_input['gmdn_definition'].dropna().str.len()
        sns.histplot(lengths, kde=True, color='#d62728', ax=ax)
        ax.set_title("Nomenclature Definition Complexity Analysis")
        ax.set_xlabel("Character Length Matrix")
    plot_outputs.append(fig)
    
    # Chart 5: Correlation Mapping Matrix of Attributes (Heatmap Layout)
    fig, ax = plt.subplots(figsize=(6, 4))
    numeric_df = df_input.select_dtypes(include=[np.number])
    if not numeric_df.empty and numeric_df.shape[1] > 1:
        sns.heatmap(numeric_df.corr(), annot=True, cmap='coolwarm', fmt=".2f", ax=ax)
        ax.set_title("Device Attribute Correlation Heatmap Matrix")
    else:
        # Fallback tracking distribution visualization
        df_input.value_counts(subset=list(df_input.columns[:2])).head(10).plot(kind='pcolor', ax=ax)
        ax.set_title("Data Sparsity/Density Overview Matrix")
    plot_outputs.append(fig)
    
    return plot_outputs

```

---

## 5. THREE ADVANCED "WOW" AI FEATURES

### Feature 1: Multi-Lingual Semantic Concept Alignment & cross-Registry Vector Mapping

The platform features an advanced vector extraction layer that bridges the language gap between the English text in the MEDIVID dataset and the bilingual entries in the TUDID database.

```
       [Raw User Request]
               |
               v
  +--------------------------+
  | Embedding Extraction     | -> (Generates Multi-Lingual Dense Vectors)
  +--------------------------+
               |
               v
  +--------------------------+
  | Vector Cosine Valuation  | -> (Calculates Semantic Distances)
  +--------------------------+
               |
               v
[Bypasses Token Blockages to Link "Forceps" with "外科手術用夾鉗"]

```

This enables the application to accurately associate terms like *"forceps"* with complex Chinese registry listings such as `“美敦力”長青植入式心臟整流去顫器` or `外科手術用夾鉗`, matching concepts by their clinical purpose rather than relying solely on exact keyword matches.

### Feature 2: Automated AI Regulatory Discrepancy & Non-Compliance Auditor

An integrated background agent evaluates new user data uploads against official global nomenclature standards to detect data inconsistencies or regulatory risks.

```
                                  [Data Stream Ingestion]
                                             |
                                             v
                             +-------------------------------+
                             |  Anomaly Detection Engine     |
                             +-------------------------------+
                              /                             \
                             v                               v
            {Outlier Detected}                       {Mismatched Risk Level}
                    |                                        |
                    v                                        v
+---------------------------------------+  +---------------------------------------+
| Flag: GMDN definition specifies       |  | Flag: TUDID lists Class 2, but global |
| "reusable", but entry is marked single|  | database identifies device as a       |
| use.                                  |  | Class 3 implantable hardware asset.   |
+---------------------------------------+  +---------------------------------------+

```

This auditor continuously checks details like single-use flags against global reuse definitions, highlighting potential issues before files are submitted to official regulatory systems.

### Feature 3: Conversational Dynamic Chart Refinement Agent

This feature updates static data dashboards into interactive, responsive analytics tools. Users can adjust or filter chart elements using simple voice or text commands:

> *"Filter out the revoked records, focus chart two entirely on Medtronic items, and update the display theme to deep jewel color palettes."*

The refinement agent processes the instruction, verifies the parameters against system security rules, adjusts the data filters, and regenerates the visuals instantly without requiring any full-page reloads.

---

## 6. COMPREHENSIVE COMPONENT VERIFICATION & INQUIRY PROTOCOLS

The following 20 validation questions are designed to review system performance, evaluate structural integrity, and ensure consistency during high-volume testing of the data pipelines:

1. How does the ingestion layer handle missing data or empty strings inside structural identification fields like `primary_di` without causing errors in the relational mapping setup?
2. What strategy does the schema processing engine use to map varied input column names, such as resolving `中文品名` into standardized target variables like `chinese_name`?
3. How can the text-to-SQL generation engine be extended to support complex conditional logic or window functions for deep data slicing?
4. What mechanisms protect the database execution engine from performance degradation when handling exceptionally large files via user copy-paste inputs?
5. How does the validation layer distinguish between benign syntax errors and intentional SQL injection strategies (e.g., stacked commands or alternate encoding vectors)?
6. When using smaller localized models like `gemini-3.1-flash-lite`, what system techniques prevent errors like schema hallucinations or improper join operations on foreign keys?
7. What criteria should the validation engine use to adjust execution plans when encountering non-standard or custom UDI issuing agencies beyond standard groups like `GS1`?
8. How does the system handle character encoding variations (such as UTF-8 vs. Big5) when processing traditional Chinese characters in fields like `許可證字號（類型）`?
9. In what ways can the system optimize execution speeds when performing full-text matches across long data descriptions like the `gmdn_definition` text blocks?
10. How can the system's export features be updated to stream very large data exports directly to user clients without hitting web server memory limitations?
11. What approaches can render large, data-dense HTML grids efficiently in client browsers when dealing with search results that return over ten thousand rows?
12. How does the deterministic plotting pipeline prevent runtime data errors when generating statistical charts from highly skewed or sparse datasets?
13. What fallback strategies ensure the data visualization engine still displays clear information if required numeric columns are entirely missing from a file upload?
14. How can the cross-registry mapping system balance performance when analyzing similarities between long English terminology definitions and shorter Chinese product names?
15. What security controls are needed to protect the local database storage environment when running multi-tenant web instances concurrently?
16. How can the automated regulatory compliance agent be updated to support changing rules as international agencies update device risk classifications?
17. What caching strategies can prevent unnecessary recalculations when multiple users run similar natural language or semantic queries?
18. How does the system design maintain separation between the AI processing layer and the core database engine to allow swapping LLM models easily without rewriting underlying logic?
19. What performance metrics should be used to monitor the accuracy of the automated text-to-SQL agent over time?
20. How can the conversational chart agent process multi-step visual requests (e.g., changing colors and filtering specific categories simultaneously) while ensuring the generated output remains predictable and secure?
