TECHNICAL_SPECIFICATION
Document Reference: MedAgent-FDA-Submit-TS-V2.0
Authors: AI Systems Engineering Core Team
Subject: High-Fidelity Custom Multi-Agent Swarm for FDA Premarket 510(k) and TFDA Submission Verification
SECTION 1: Executive Overview & System Design Philosophy
1.1 Document Purpose & Scope
This Technical Specification details the architecture, design patterns, and systemic state orchestration of MedAgent FDA-Submit, a clinical-grade multi-agent consultative platform. This platform is custom-built to aid regulatory affairs (RA) professionals, clinical trial specialists, biomedical engineers, and legal compliance officers in reviewing medical device submissions. It targets US FDA Premarket Notification [510(k)] and Taiwan Food and Drug Administration (TFDA) regulatory pathways, standardizing the verification of documentation against structural checklists representing major international standards (e.g., ISO 13485, ISO 14971, ISO 14155, ISO 10993, and IEC 60601).
Rather than relying on a monolithic AI agent to read complex biomedical specifications, MedAgent partitions clinical judgment. It employs an isolated, cooperative group of analytical agents (the Agent Swarm), each operating within strict regulatory boundaries. This specification outlines how users can dynamically expand, configure, provision, and audit this swarm by introducing custom agents backed by distinct instruction sets loaded from .md files (i.e., skill.md). This architecture operates within sandboxed iframe boundaries without exposing sensitive parameters or breaking browser context.
1.2 System Objectives
Decoupled Specialization: Isolate regulatory concerns (e.g. Clinical Statistics, Risk Mitigation, Labeling Standards) into independent intelligence units to avoid hallucination cascades.
Dynamic Customization (User-Generated Swarm): Allow end-users to dynamically bootstrap custom regulatory reviewers by defining system role guidelines and dragging/dropping custom checklist files (skill.md).
Robust Bilingual Operability: Seamlessly manage active state, localization (en-US and zh-TW), visual styling presets (sleek, elegant, professional), and live outputs on a single unified canvas.
Execution Isolation and Sandbox Fault-Tolerance: Safely manage state serialization across client boundaries and handle iframe storage container exceptions gracefully.
SECTION 2: Unified Multi-Agent Heuristics & Swarm Architecture
code
Code
=======================================================
                  |                  Client GUI                         |
                  |  (Single-Page Canvas with Dark/Light Styles)        |
                  =======================================================
                                             |
                  +--------------------------+--------------------------+
                  |                                                     |
         [YAML Fetch Engine]                                  [Custom Agent Config]
                  |                                                     |
                  v                                                     v
      =========================                             =========================
      |   agents.yaml Parser  |                             | Custom JSON Loader    |
      |   (Dynamic fetch)     |                             | (LocalStorage Guard)  |
      =========================                             =========================
                  |                                                     |
                  +--------------------------+--------------------------+
                                             |
                                             v
                         =========================================
                         |        Consolidated Agent Swarm       |
                         |   (State Sync with fallbacks)         |
                         =========================================
                                             |
                       +---------------------+---------------------+
                       |                     |                     |
                       v                     v                     v
                [Lead Agent]         [Clinical Agent]       [Risk Agent]      ...[Custom Agents]
                (CFR 807 Skill)      (ISO 14155 Skill)      (ISO 14971 Skill)    (User Uploaded Skills)
The system models its multi-agent intelligence layer as a stateful blackboard system. When triggered, agents evaluate the draft submissions on a per-module basis according to their functional checklists.
2.1 Core Agent Layout & Structure
A single-agent instance in the MedAgent pipeline is modeled using the following strict interface format:
code
TypeScript
type AIModel = 'gemini-3.1-flash-preview' | 'gemini-3-flash-preview';

interface Agent {
  id: string;                                  // Unique identifier (UUID or derived)
  name: Record<Language, string> | string;     // Localized or generic literal name
  enabled: boolean;                            // Swarm participation toggle
  model: AIModel;                              // AI foundation engine assignment
  prompt: string;                              // Injected system role prompt
  role?: Record<Language, string> | string;    // Operational role details
  expertise?: Record<Language, string> | string;// Specific capability vectors (e.g., ISO 14971, ISO 10993)
  skillName?: Record<Language, string> | string;// Structural checklist label
  skillContent?: string;                       // Markdown contents of the injected skill
  isCustom?: boolean;                          // Boolean flag indicating runtime provision
}
2.2 The Default Agent Swarm (agents.yaml)
To establish a rigid control plane, the system fetches pre-configured agents from a relative /agents.yaml file on boot. If the physical fetch is blocked or the file is missing, the system catches the error and mounts hardcoded fallback structures in state. This ensures a durable runtime environment.
The default layout defines three archetypal roles:
Regulatory Lead (id: lead): Specializes in 21 CFR Part 807 General Competency. Coordinates overall submission readiness, substantial equivalence outlines, and FDA forms conformance.
Clinical Data Analyst (id: clinical): Specializes in ISO 14155 Clinical Protocol Audits. Evaluates biostatistical sample sizing, pediatric cohort exclusions, and clinical endpoints.
Risk Management Specialist (id: risk): Specializes in ISO 14971 Risk-Benefit analysis. Cross-examines hazards associated with hardware, software, user error, and device failure modes.
2.3 Dynamic Custom Agent Integration Block
When an end-user defines a custom agent, the client constructs a new Agent object with a generated timestamp identifier (custom-Date.now()). The custom agent's specifications can inherit a detailed checklist from a .md upload. The system parses this file instantly and binds the contents as raw string variables into the agent's memory layout (skillContent).
SECTION 3: Detailed Data Flow and Stateful Orchestration
The application utilizes a single unified state manager running under a React context tree. Since the environment operates within a sandboxed virtual frame, specific storage mechanisms must be hardened against security constraints.
code
Code
[System Boot]
     |
     v
[Fetch /agents.yaml] ---> Successful? ----> YES ---> [Parse YAML using js-yaml] ---> [Warm Default State]
     |                                 |
     | (Network/File Blocked)          +--> NO  ---> [Load Fallback JS Records] ----+
     v                                                                              |
[Load Custom Agents from LocalStorage] ---> Exception Safeness?                     |
     |                                                                              |
     | (Iframe Restrictions Lifted?)                                                |
     +--> YES ---> [JSON Parse and Merge] ------------------------------------------+
     |                                                                              |
     +--> NO  ---> [Warm Empty Array & Log Safe Warning]                            |
                                                                                    v
                                                                     [Construct Swarm Blackboard]
3.1 Fetch and Parse Pipeline (agents.yaml)
The YAML parsing pipeline relies on the modular js-yaml library. Using Vite ES Module path resolution, import { load as parseYaml } from 'js-yaml' avoids bundling issues during development. The fetching sequence is wrapped in an async function:
code
TypeScript
const response = await fetch('/agents.yaml');
if (response.ok) {
  const yamlText = await response.text();
  const parsed = parseYaml(yamlText) as { agents: any[] };
  // Mapping checks are handled with deep structural validation
}
This prevents malformed external definitions from crashing the startup thread. If any dynamic conversion fails, fallback primitives immediately restore application state.
3.2 Secure Client-Side Persistence
In sandboxed execution models, browser access to localStorage may trigger a SecurityError (e.g., when the iframe belongs to a different origin than the host parent window). MedAgent prevents white-screen faults by wrapping all setItem and getItem transactions in isolated try/catch clauses:
code
TypeScript
// Safe LocalStorage Read Pattern
const [customAgents, setCustomAgents] = useState<Agent[]>(() => {
  try {
    const saved = localStorage.getItem('medagent_custom_agents');
    return saved ? JSON.parse(saved) : [];
  } catch (e) {
    console.warn('LocalStorage reads disabled in current frame/sandbox.', e);
    return [];
  }
});
code
TypeScript
// Safe LocalStorage Write Pattern
try {
  localStorage.setItem('medagent_custom_agents', JSON.stringify(updated));
} catch (e) {
  console.warn('LocalStorage write disabled in current frame/sandbox.', e);
}
This ensures that even if local persistent state is lost during sub-session refreshes, the core interface remains fully operational and functional.
SECTION 4: Deep-Dive into the 3 "Wow AI" Features
To elevate MedAgent from a static mock-up tool to a powerful AI review platform, we introduce three innovative AI features. Below is their technical design, data structures, and algorithmic architectures.
FEATURE 1: Cross-Agent Regulatory Conflict Resolver (Semantic Reconciliation)
A. Technical Concept
In complex submissions, different expert agents may make contradictory recommendations. For instance:
The Risk Specialist (risk) insists that a pneumatic cuff inflation system requires an immediate mechanical pressure-relief backup valve (per ISO 14971).
The Lead Regulatory Coordinator (lead) highlights that adding this valve alters the physical predicate matching parameters of the predicate device (K152345), complicating the equivalent model path under 21 CFR 807.87.
The Cross-Agent Regulatory Conflict Resolver parses individual agent findings using semantic similarity and constraint logic. It checks for engineering contradictions and produces a reconciled, risk-prioritized action plan.
B. Architectural Data Flow & Formula
Extract Context Vectors: Let 
 be the output text block generated by Agent 
. We build semantic tokens and check for logical intersections of critical vocabularies (e.g., "valve", "warning", "clinical study").
Conflict Matrix Calculations: The system builds a conflict mapping table 
 of dimension 
 (where 
). Each element 
 represents a potential conflict index.
Conflict Probability Verification: Using token proximity matching, we calculate the contradiction probability:

Where:
 is the sigmoid function.
 is the weight of the 
-th indicator keyword (e.g., "must restrict" vs "must implement").
 is a binary correlation function indicating if a specific clinical noun or regulation is shared with differing instructions.
Reconciliation Prompt Formulation: If a potential conflict is identified (
), the system combines both recommendations and sends them to a dedicated consolidation function. This function serves as an AI Mediator to draft three alternative paths (regulatory-first, safety-first, or hybrid-reconciliation).
code
Code
[Agent 1 Output (Risk)] ------+
                             |
                             v
                 =========================
                 |  Semantic Contradiction| ---> Flagged? ---> YES ---> [Execute AI Mediator]
                 |     Mapping Engine     |                                    |
                 =========================                                     v
                             ^                                      [Generate Multi-Path Options]
                             |                                      1. Safety-Prioritized
[Agent 2 Output (Lead)] -----+                                      2. Substantial-Equivalence-First
                                                                    3. Integrated Reconciliation
C. Interface Implementation Blueprint
An interactive Conflict Board panel resides beneath the general reports section. This board highlights highlighted conflicts in orange/red banners, showing the opposing agent avatars, side-by-side arguments, and an interactive "Apply Resolution" button that updates the submission draft.
FEATURE 2: Interactive 21 CFR Gap Heatmap & Bento-Tree Grid
A. Technical Concept
Instead of rendering audit findings in linear, text-heavy reports, the Interactive Gap Heatmap transforms raw regulatory gap indices into a responsive, high-fidelity visual grid.
This component maps the standard 21 CFR parts (such as 807, 801, 803, and 820) to structured tiles. The sizes and color gradients of these tiles vary dynamically based on the frequency and severity of the issues flagged by the active agent swarm.
B. Algorithmic Grid Mapping
We define the gap weight of a given CFR chapter 
 using:

Where:
 represents risk severity (e.g., Critical = 5, Warning = 3, Draft Outline = 1).
 represents the number of analytical errors identified in that specific section of the document.
The client dynamically calculates these weights and organizes a CSS Grid container using the standard Squarified Treemap Layout algorithm. Highly compliant sections (weight 
) glow with deep cyan/emerald hues, while missing descriptions or unvalidated trials trigger intense warnings with high-contrast amber/rose backgrounds.
code
Code
+----------------------------------------+---------------------------------------+
| 21 CFR Part 807 (Ready)                | 21 CFR Part 801 (Check Label)         |
| [Status: Emerald (Ready)]              | [Status: Amber (Warning)]             |
| Overall Conformity Index: 96%          | Missing Pediatric Safety Labels       |
| Area: 55%                              | Area: 25%                             |
|                                        |                                       |
+----------------------------------------+---------------------------------------+
| 21 CFR Part 820 (Quality System)       | 21 CFR Part 803 (Medical Reporting)   |
| [Status: Zinc (Draft)]                 | [Status: Rose (Critical)]             |
| Area: 12%                              | Area: 8%                              |
+----------------------------------------+---------------------------------------+
C. Interactive Refinement Pipeline
Clicking on any segment of the Treemap reveals a side-drawer displaying:
Section Summary: Detailed explanation of the compliance gap.
Citations: Relevant code excerpts (e.g., 21 CFR 807.87(f)).
Remediation Suggestion: A preview block of recommended text that can be instantly appended to the active section of the submission.
FEATURE 3: Automated Predicate Matchmaker & Equivalence Modeler
A. Technical Concept
To secure a 510(k) clearance, a device must demonstrate Substantial Equivalence (SE) to a legally marketed predicate device. This matchmaker feature processes the uploaded Device Description & Indications to identify key technical specs, evaluate clinical indicators against comparable systems, and model similarity scores.
B. Similarity Scoring Engine & Mathematical Modeling
The model measures the mathematical similarity of a candidate device 
 and a predicate device 
 across discrete technical parameters (Pneumatic Pressure 
, Software Class 
, Battery Power 
):
Where:
 measures the vector similarity of numeric features.
 represents the semantic distance between the target indications (computed using Jaccard Similarity of processed medical terms).
 and 
 are control coefficients balancing physical features versus indications for use.
code
Code
[Raw PDF Description / JSON Specifications]
                         |
                         v
           [Parameter Extraction Layer]
                         |
  +----------------------+----------------------+
  |                                             |
  v                                             v
[Physical Characteristic Code: V_cand]       [Clinical Intended Use Terms: T_cand]
  |                                             |
  +----------------------+----------------------+
                         |
                         v
       [Consolidated Reference Similarity Algorithm]
                         |
                         v
     =======================================
     |  SE-Score Matrix Estimation Engine  |
     =======================================
                         |
                         +--> Score >= 85% -> Legally Viable Predicates Listed
                         +--> Score < 85%  -> Critical Variance Warning Triggered
C. User Interface Implementation
This tool adds a Substantial Equivalence Modeler to the primary viewing section. This panel presents candidate predicates along with similarity metrics, sliding parameter comparisons, and an auto-generated Substantial Equivalence Comparison Table ready for inclusion in Section 12 of the submission.
SECTION 5: Dynamic Guidelines (Skill.md) Parsing & Sandbox Isolation Layer
Executing client-side script parses guidelines securely within an isolated execution boundary. This section discusses storage schemas, parser configurations, and recovery paths.
5.1 Sandbox Constraints Configuration
Since the applet runs in an environment with strict frame permissions, file uploads and code-level operations must use standard, compliant Web APIs.
Audio/Camera Sandbox: Marked optional in metadata.json so as not to prompt the user unnecessarily.
No Native Shell Execs: The code executes exclusively in safe runtime contexts. It reads raw metadata from memory buffers using FileReader() without requiring direct file system mounts or backend paths.
5.2 Dynamic Skill Injector Implementation
When a custom agent is created, the system reads the markdown content from the uploaded file and stores it in the browser's memory scope. When the agent joins the active swarm:
Context Loading: The agent's checklist (skillContent) is appended directly to its standard instructions.
Log Integration: System components trace which custom instruction sets were parsed, outputting events to the live logs (e.g. "[System] Created and activated custom agent: Biosafety Expert").
Blackboard Update: The system recalculates overall regulatory readiness scores, incorporating these custom checklist guidelines into the report generation pipeline as shown on the interface.
SECTION 6: Future Capabilities & Scale Vector Analysis
To scale MedAgent beyond client-side rendering, developers can expand the system across several architectural vectors:
code
Code
[Candidate Workspace]
        |
        +-----> API Gateway Proxy (Port 3000)
                     |
                     +-----> Server-Side Express Service (server.ts / Node.js)
                                  |
                                  +-----> Secure Gemini SDK (@google/genai)
                                  |            * Gemini 1.5 Pro (Deep Reasoning)
                                  |            * API Keys masked via env
                                  |
                                  +-----> Database Engine (Firestore Integration)
                                               * Persistent Multi-Session History
Durable Backend Integration (Full-Stack Express & Node): Route complex semantic checks through an internal server.ts file running on Port 3000. This server proxies requests securely to Google GenAI models utilizing the standard GEMINI_API_KEY stored secret.
Persistent Historical Databases (Cloud Firestore): Move custom agent configurations and submission histories to collections, eliminating browser storage limits and enabling multi-session recovery.
Active Document Parsing (Structured OCR): Integrate PDF parsers to extract text directly from actual FDA form drawings and schemas.
SECTION 7: 20 Comprehensive Follow-Up Questions
To further specialize the MedAgent platform, we present twenty follow-up questions for regulatory planning and design selection:
Swarm & Agent Design Questions
Agent Prioritization: Should default agents (e.g., lead) have a veto over custom agents, or should the swarm resolve conflicts via general consensus?
Persona Tuning: For custom agents, do we need presets for national standards (e.g., PMDA in Japan, CE MDR in Europe, TFDA in Taiwan)?
Self-Correction Cycles: Should agents critique each other's work over multiple cycles to refine findings before presenting them to the user?
Security Boundaries: How should the platform handle proprietary prompt templates so they remain hidden from end-user inspection?
Technical & File Specification Questions
Multi-Language Extensibility: How should the system handle translations for custom agents if the platform language switcher changes?
Checklist Serialization Format: Should custom agents export in JSON format or standardized Markdown with YAML front-matter?
Dynamic Web Worker Compilation: Should we transition the YAML parsing and verification pipelines to background Web Workers to prevent UI lag with large files?
Strict Semantic Validation: How should the system respond to malformed markdown formatting in uploaded checklist files?
AI & LLM Engine Optimization Questions
Model Selection Routing: Under what conditions should specific agents route requests to larger, higher-capacity models (e.g. Gemini Pro) rather than quicker, standard models?
Context Chunking Design: What token-splitting and retrieval strategy should we use when an uploaded draft exceeds standard context windows?
Checklist Conformity Guardrails: How can we ensure custom system prompts do not override critical, hardcoded FDA requirements?
Vector Space Refinements: How should the system generate semantic representations for medical vocabularies (e.g., MeSH terms or SNOMED CT)?
User Experience & Interactive Dashboard Questions
Heatmap Grid Preferences: Should the 21 CFR Gap Heatmap let users configure custom compliance metrics, or stick to the automated, risk-weighted scale?
Direct Suggestion Insertion: When resolving a conflict, should the system provide a side-by-side git-style diff of the changes before applying them?
Collaborative Review Flows: Should the platform support sharing specific agent logs and reports directly with team members via secure share parameters?
Custom Sound Settings: For long-running swarm operations, should we add optional audio cues or desktop notifications when an execution finishes?
Regulatory Compliance & Verification Questions
Substantial Equivalence Databases: How can we integrate public FDA API endpoints (e.g., accessGUDID or openFDA databases) in server-side mock-ups to search real predicate devices?
ISO 13485 Conformity Checks: Which document standards represent the logical next expansion module for the default agent suite?
Digital Signature Proofs: Should cleared reports include cryptographic checksums to verify that nobody modified the document after the swarm audit?
Audit Trail Verification: How should we log previous agent findings so regulatory authorities can verify how a design changed over time?
CONCISE OVERVIEW OF COMPLETED IMPLEMENTATIONS
I have improved the multi-agent orchestration of the MedAgent FDA-Submit platform to deliver a highly interactive client experience:
Dynamic Custom Agent Engine: Created a stateful, fully localized modal workspace (Custom Agent Builder) allowing users to dynamically define clinical role profiles, select target models, define custom checklist guidelines, and upload custom markdown files (skill.md).
State Alignment & LocalStorage Safeguard: Wrapped storage reads and writes in defensive try-catch statements to prevent any white-screen problems on sandboxed environments.
Dynamic Swarm Simulator and Action Logs: Configured runtime simulation logs to stream live evaluations reflecting the exact roles, guidelines, and skills of the user's custom agent swarm.
Skill Inspector System: Implemented an on-screen checklists viewer permitting users to audit, adjust, and re-tune loaded agent files dynamically without having to reload the app.
