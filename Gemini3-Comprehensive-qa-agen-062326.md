TECHNICAL ARCHITECTURE SPECIFICATION
Project Identifiers: WOW Agentic Workspace & AI Note Keeper v3.5.0
Target Environment: Cloud Run Container Sandbox (Port 3000, Nginx Reverse Proxy, Server-Side Node + Express, Client-Side Vite + React 19)
Author: Google AI Studio Coding Agent
Status: Architecture Proposal & Complete Technical Specification
1. EXECUTIVE SUMMARY
The WOW Agentic Workspace & AI Note Keeper is a unified, high-density decision cockpit designed for multi-agent execution, predictive supply chain management, and intelligent knowledge synthesis. By blending direct YAML-driven agent orchestration with spatial supply chain intelligence (GIS, Purchase Records, Distribution Channels), the system allows operators to run simulations, evaluate risks, generate multi-stage agent workflows, and compile interactive workspace notes.
This specification details a complete, production-ready blueprint for upgrading the existing application. It retains 100% of the current code features (the custom YAML agent compilers, Pantone theme rotators, CSV supply chain parsing, split Note Keepers, and simulation tabs) while introducing three revolutionary "Wow" features:
Dynamic Visual Node Graph Agent Orchestrator (Canvas-based Canvas Flow)
Digital Twin Supply Chain Simulation Engine & Predictive Risk Modeler (Gemini Schema Grounding)
Semantic Workspace Memory & Semantic Knowledge Graph RAG Engine
This document serves as the absolute technical reference, detailing layout mechanics, state management patterns, API designs, algorithmic paths, and deployment considerations. It concludes with exactly 20 comprehensive follow-up questions designed to probe project requirements, backend configurations, and product scaling paths.
2. SYSTEM ARCHITECTURE & DATA FLOW
The application utilizes a robust, full-stack architecture tailored for low-latency state changes, heavy client-side computations (such as CSV parsing, GIS mappings, and canvas rendering), and secured server-side AI processing to obscure sensitive API keys.
code
Code
+-------------------------------------------------------------+
       |                     CLIENT BROWSER (PORT 3000)               |
       |                                                             |
       |  +--------------------+   +------------------------------+  |
       |  |   App.tsx Controller|   |   React State Core           |  |
       |  |                    |   |                              |  |
       |  |   - Active Tab     |---|   - GIS / CSV Records        |  |
       |  |   - Active Pantone |   |   - Agent Run Queue State    |  |
       |  +--------------------+   +------------------------------+  |
       |             |                            |                  |
       |             v                            v                  |
       |  +--------------------+   +------------------------------+  |
       |  |   Core Modular Tabs|   |   Proposed "Wow" Components" |  |
       |  |                    |   |                              |  |
       |  |   - Home / Settings|   |   - Visual Node Graph        |  |
       |  |   - Runner Tab     |   |   - Gemini Digital Twin      |  |
       |  |   - Simulation Tab |   |   - Semantic Graph Note RAG  |  |
       |  |   - Note Keeper    |   |                              |  |
       |  +--------------------+   +------------------------------+  |
       +---------------------------------------------^---------------+
                             |                       |
                  JSON Rest & SSE Protocol           | Real-time Stream
                             |                       |
=============================v=======================|==============
       +----------------------------------------------+--------------+
       |                   EXPRESS BACKEND (PORT 3000)               |
       |                                                             |
       |  +-----------------------------+                            |
       |  |      server.ts Router       |                            |
       |  |                             |                            |
       |  |  - /api/keys-status         |                            |
       |  |  - /api/agent/compile       |                            |
       |  |  - /api/agent/stream-run    |                            |
       |  |  - /api/twin/forecast       |                            |
       +--+-----------------------------+----------------------------+
                         |
                         | HTTPS with @google/genai SDK
                         v
       +-------------------------------------------------------------+
       |                  EXTERNAL CLOUD API LAYERS                  |
       |                                                             |
       |   - Google Gemini 3.1 Flash / Ultra API Services            |
       |   - Developer Keys (Gemini, OpenAi, Anthropic, xAi)         |
       +-------------------------------------------------------------+
2.1 Backend Decoupling and Secret Hydration
The Node/Express backend (server.ts) initializes and coordinates the client-side single page app (SPA) assets in production (serving the built files out of dist/) and proxies all AI workload integrations. The server accesses sensitive keys directly from process.env (e.g., process.env.GEMINI_API_KEY). At no point are developer keys transmitted down to the browser.
In development, the Vite server middleware binds to the Express app via createViteServer, maintaining Hot Module Replacement (HMR) for instant source validation while avoiding cross-origin resource sharing (CORS) or proxy-redirection drift.
3. DECONSTRUCTION OF ORIGINAL WORKSPACE COMPONENTS
To preserve the utility of the previous version, we first break down the architecture of the existing modules to ensure complete backwards-compatibility.
3.1 Agent Core Engine & YAML Parsing System
The system features a custom, lightweight compiler that reads a structured agent config file represented in YAML formats. This YAML describes a sequential pipelined network of conversational agents with specific parameters.
code
Yaml
- id: researcher
  name: Research Specialist
  role: Supply Chain Analyst & Tech Sorter
  systemInstruction: "You are an expert researcher..."
  defaultModel: gemini-3.1-flash-lite
  temperature: 0.7
  maxTokens: 4000
Parsing Algorithm:
Lexical Analysis: Splits the YAML string line-by-line.
Node Assembly: Dynamically scans for array elements (- id: or id:) and constructs Agent interfaces comprising properties like id, name, role, systemInstruction, defaultModel, temperature, and maxTokens.
Validation: Resolves structural hazards such as empty strings, missing identifiers, or invalid floating-temperature settings, surfacing errors through reactive string arrays on the yamlWarnings state.
Inputs Orchestration: Instantiates reactive input arrays on the dashboard matching each parsed runner input model, priming the runtime pipeline.
3.2 Supply Chain Datasets (GIS, Purchase, Distribution)
A vital core component is the embedded smart supply chain monitoring dashboard. This module is powered by static datasets found in /src/data/defaultDatasets.ts and reactive state buffers managed by the csvParser utility.
GIS Record Layer (GISRecord): Captures coordinates of distribution facilities, tracking asset attributes (e.g., facility ID, regional hubs, localized delays, overall operating throughput, hazard status).
Purchase Record Layer (PurchaseRecord): Documents order numbers, global raw supplier origins, freight costs, shipment volumes, unit counts, and lead-time constraints.
Distribution Network Layer (DistributionRecord): Traces shipping transit metrics, transit modes (Air, Ocean, Rail, Road), delivery states, load factors, and total carbon footprint.
The system includes helper routines parsePurchaseCSV and parseDistributionCSV that dynamically parse customized user uploads. If a file is dropped in, the system adapts, shifting state variables, drawing fresh geographical graphs, updating stress calculations, and shifting datasetSource to "custom".
3.3 Multi-Tab Core Layout & Reactive UI Core
The visual cockpit contains multiple specialized panels that are hot-swapped using the local state variable activeTab to achieve immediate tab switching without loading visual layout frames:
Home ("home"): Contains a layout describing the WOW Orchestrator features and key operation alerts.
Runner Tab ("runner"): Provides side-by-side terminal controls where parsed agents execution feeds stream sequentially.
Dashboard ("dashboard"): Aggregates business data and renders custom visual metric curves using recharts.
Notes Tab ("notes"): Side-by-side split screen supporting rich markdown outputs and AI-backed transformations.
Datasets Tab ("datasets"): Grid layout listing, editing, resetting, and uploading new CSV configurations.
GIS Tab ("gis"): Renders facility locations and connections complete with overlay search sliders and hazard indicators.
Audit Tab ("audit"): Form-based panel evaluating inventory matching metrics and displaying compliance scores.
Simulation Tab ("simulation"): Allows operators to set severity parameters, drop visual elements, and test overall network tolerance.
3.4 Note Keeper Markdown Workspace & AI Services
The Note Keeper operates as a full-featured markdown workspace containing custom integration buttons for semantic operations:
Split Mode / Edit Mode / Preview Mode: Powered by custom viewport dividers.
Tone Adjuster & Text Extenders: Routes the active note buffer through a server-side Gemini request, applying styles such as "executive", "technical", or "creative".
Fusion Magic: Merges secondary information or CSV snippets with the active note buffer, performing contextual consolidation using Gemini's reasoning layers.
Entity & Graph Extractor: Dynamically parses notes to isolate semantic connections and outputs interactive relational graphs.
4. THREE NEXT-GENERATION "WOW" CORE FEATURES
The proposed features augment the original functionality by turning static visual representations into highly interactive pipelines.
code
Code
+-----------------------------------------------------------------------------------------+
|                                    WOW WORKSPACE DESKTOP COCKPIT                         |
+-----------------------------------------------------------------------------------------+
| [ Home ] [ Agent Node Flow ] [ Supply Chain GIS ] [ Notes & RAG ] [ Twin Simulation ] |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  +---------------------------------------------------------------+                       |
|  |             1. VISUAL AGENT FLOW CANVAS DIAGRAM              |                       |
|  |                                                               |                       |
|  |     [ Researcher Node ] ===== (JSON Flow) =====> [ Analyst ] |                       |
|  |             |                                         |       |                       |
|  |             v                                         v       |                       |
|  |       { Stream Log }                            { Analytics } |                       |
|  +---------------------------------------------------------------+                       |
|                                                                                         |
|  +-------------------------------------+   +------------------------------------------+ |
|  |    2. SUPPLY CHAIN GIS & TWIN       |   |      3. KNOWLEDGE NOTE GRAPH (RAG)      | |
|  |                                     |   |                                          | |
|  |  +-------------------------------+  |   |    [Note #1] <---> (Entity: "Suez Hub")   | |
|  |  | Facilities (GIS Markers)      |  |   |       ^                 |                | |
|  |  | Sim Disruption: Red Sea Surge |  |   |       |                 v                | |
|  |  | Gemini Recommendation: RouteB |  |   |       v           [Note #2]              | |
|  |  +-------------------------------+  |   |  - Prompt: "Check shipping incidents"    | |
|  +-------------------------------------+   +------------------------------------------+ |
+-----------------------------------------------------------------------------------------+
4.1 Feature 1: Dynamic Visual Node Graph Agent Orchestrator (Canvas-Based Canvas Flow)
Rather than executing agent runs in a basic terminal list, this feature transforms agent composition and sequential execution into a visual layout.
Operational Overview:
An HTML5 Canvas container reads parsed agents from agentsYaml and maps them as floating interactive nodes. Connections between nodes represent semantic streams (output from Agent A pipes directly into the system prompt of Agent B).
code
Code
+--------------------------------------------+
|            Agent Node (Researcher)         |
|  +--------------------------------------+  |
|  | Output Token Rate: 125 t/s           |  |
|  | Temp: 0.7  | Model: Flash-Lite       |  |
|  +--------------------------------------+  |
|  O (Input Anchor)      (Output Anchor) O---|======> [Analyst Node]
+--------------------------------------------+
Visual Mechanics:
Position Matrix: Every agent in the pipeline is given an 2D coordinate 
. The layout positions them left-to-right based on their array index.
Reactive Hooks: Mouse events (MouseDown, MouseMove, MouseUp) track node drags. Lines representing data streams are rendered as responsive cubic bezier paths between anchors.
Live Stream Glow: When an agent task executes, the corresponding canvas boundaries pulse using specific Pantone system colors (currentStyle.glow). Visual "data pulses" traverse along curves toward target nodes, reflecting output token rates.
Visual Editor Interface: Operators can double-click a node to focus its agent settings (Temperature, Model, Max Tokens) in a slide-out panel, or drag curves between nodes to rewire pipeline parameters.
4.2 Feature 2: Digital Twin Supply Chain Simulation Engine & Predictive Risk Modeler
The original simulation tab provides static parameters for cargo delays. This feature integrates a true "Digital Twin" simulation layer. It runs Monte Carlo algorithms in the background to model risk propagation across assets and geographical facilities, drawing contextual reasoning directly from the Gemini API.
Operational Overview:
When a disruption event is active (e.g., "Pacific Sea Lane Shutdown"), the system feeds the coordinates and capacities of all GIS Facilities, freight lanes, and current purchase records into a structured predictive model.
code
Code
+---------------------------------------------------------+
       |                  Simulation Tab Variables               |
       |  - Selected Scenario: Major Port Congestion             |
       |  - Disruption Coordinates: 34.0522, -118.2437           |
       |  - Impact Corridor Radius: 450 km                       |
       +---------------------------------------------------------+
                                    |
                                    v
       +---------------------------------------------------------+
       |             Digital Twin Risk Propagator (Engine)       |
       |  - Computes spatial overlaps across facilities          |
       |  - Identifies bottleneck nodes in distribution paths     |
       |  - Simulates demand backlogs and inventory stockouts    |
       +---------------------------------------------------------+
                                    |
                                    v
       +---------------------------------------------------------+
       |             Gemini Structured Decision Engine           |
       |  - Structured outputs schema requests alternatives      |
       |  - Generates rerouting options & freight rate impact     |
       +---------------------------------------------------------+
Analytical Breakdown:
Spatial Overlap Calculations: Compares the coordinate radius of simulated impact events against active GIS markers:
Bottleneck Node Isolation: Dynamically parses your active purchase dataset (purchaseRecords) and ranks affected facilities by total throughput.
Structured AI Routing Decisions: Send structured JSON queries to the Gemini model containing affected facilities. Request the model return structured remediation strategies including:
Alternation sea-rail transport plans.
Estimated increase in overall transit times.
Alternative supplier facilities outside the hazard zone.
Interactive Visual Overlays: The GIS mapping overlay is updated in real time, coloring affected facilities deep crimson, drawing dynamic rerouting paths, and overlaying predictive throughput curves.
4.3 Feature 3: Semantic Workspace Memory & Semantic Knowledge Graph RAG Engine
The original "Markdown Note Keeper Extraction" displays simple graphical networks. This feature introduces a comprehensive client-side Semantic Memory Vault. It extracts structural connections from all saved markdown notes and links them to supply chain entities.
Operational Overview:
The RAG (Retrieval-Augmented Generation) memory manager catalogs files, GIS facilities, and active agent execution runs. It exposes a unified index that users can query to locate conceptual references.
code
Code
+------------------------------------+
                    |       Semantic Note Keeper         |
                    |  - marketing_brief.md              |
                    |  - audit_log_q4.md                 |
                    +------------------------------------+
                                      |
                                      v
                    +------------------------------------+
                    |    Structured Entity Extraction    |
                    |  - Facility ID: FAC-01             |
                    |  - Region: LA Port                 |
                    +------------------------------------+
                                      |
                                      v
+------------------+   +------------------------------+   +-------------------+
| GIS Facility Tab |<==|   Dynamic Semantic Linker    |==>| Agent Run Outputs |
|  - Coordinates   |   |  - Matches ID to local text  |   |  - Run histories  |
|  - Throughputs   |   |  - Maps spatial connections |   |  - Log summaries  |
+------------------+   +------------------------------+   +-------------------+
Analytical Breakdown:
Unified Graph Network: Builds an in-memory knowledge representation comprising Nodes (Entities) and Edges (Relationships).
Local Term Frequency Scanning: Scans text blocks for key system terms (e.g., IDs, facility names, transit bottlenecks, agent identifiers) and links matching notes to spatial coordinates.
Contextual RAG Retrieval: When a user queries some context (e.g., "Show logistics issues described in Q4 notes"), our background agent executes a local cosine vector ranking simulation or triggers a server-side extraction pipeline to aggregate relevant markdown files.
Note Linker Tooltips: In Note Keeper preview screens, identified entities (such as "FAC-01" or "Researcher Agent") are styled as clickable inline links. Clicking a link shifts active views to highlight the target element's status page.
5. RE-ALIGNING THE FILE LAYOUT MATRIX
To accommodate the unified design, we map our files as clean modular structures:
/metadata.json: Contains workspace names and frame capabilities. Must be populated with real app details.
/tsconfig.json: Defines compilation options.
/vite.config.ts: Configures plugins and aliases.
/package.json: Organizes dependencies and runtime scripts.
/server.ts: Decouples and secures critical model operations. Maintains production fallbacks.
/src/main.tsx: Standard client-side React entrypoint.
/src/index.css: Imports fonts and declares visual color variables.
/src/types.ts: Decouples data types to prevent visual circular reference failures.
/src/App.tsx: Central controller coordinating core state parameters.
/src/data/defaults.ts: Contains style matrices, YAML configs, and system logs.
/src/data/defaultDatasets.ts: Contains fallback supplier GIS structures.
/src/lib/csvParser.ts: Lightweight parser routines translating drops into objects.
/src/components/HomeWowAi.tsx: Layout dashboard emphasizing system alerts.
/src/components/DatasetTab.tsx: CSV uploader displaying data sheets.
/src/components/GisTab.tsx: Maps facility vectors and risk boundaries.
/src/components/AuditTab.tsx: Facilitates inventory balancing runs.
/src/components/SimulationTab.tsx: Disruption scenario configuration workbench.
/src/components/VisualNodeGraph.tsx: Visualizes dynamic agent flow pipelines on canvas.
/src/components/DigitalTwinPanel.tsx: Houses predictive modeling tables and route analyses.
/src/components/SemanticMemorySidebar.tsx: Orchestrates notes and dynamic entity mappings.
6. IN-DEPTH TECHNICAL SPECIFICATIONS & IMPLEMENTATION BLUEPRINTS
This section provides granular code architectures, interface declarations, and algorithmic flow charts for the advanced systems.
6.1 State Management & Central Types (types.ts)
To prevent the visual "circular dependency" hell in large-scale React files, we isolate and standardize our core interfaces in a centralized /src/types.ts registry.
code
TypeScript
// Shared Types & Interfaces Reference
export type TabName = 
  | "home" 
  | "runner" 
  | "dashboard" 
  | "notes" 
  | "studio" 
  | "settings" 
  | "datasets" 
  | "gis" 
  | "audit" 
  | "simulation"
  | "nodegraph" // Added Node Graph view
  | "twin";     // Added Digital Twin view

export interface PantoneStyle {
  id: string;
  name: string;
  primary: string;   // Tailwinds hex
  secondary: string;
  accent: string;
  darkBg: string;
  lightBg: string;
  glassBg: string;
  glow: string;      // RGB Glow string for Canvas Glow drops
}

export interface Agent {
  id: string;
  name: string;
  role: string;
  systemInstruction: string;
  defaultModel: string;
  temperature: number;
  maxTokens: number;
}

export interface NodePosition {
  id: string;
  x: number;
  y: number;
  vx: number;
  vy: number;
  width: number;
  height: number;
  isDragging: boolean;
}

export interface NodeEdge {
  id: string;
  source: string;
  target: string;
  label?: string;
  isActive: boolean;
  pulseProgress: number; // 0 to 1 for visual packet flow animations
}

export interface GISRecord {
  facilityId: string;
  facilityName: string;
  latitude: number;
  longitude: number;
  status: "Nominal" | "Alert" | "Critical";
  throughput: number; // in tons / day
  headquarters: string;
}

export interface PurchaseRecord {
  orderId: string;
  supplier: string;
  country: string;
  cost: number;
  quantity: number;
  destinationFacility: string;
  transitDays: number;
}

export interface DistributionRecord {
  routeId: string;
  origin: string;
  destination: string;
  carrierMode: "Air" | "Ocean" | "Rail" | "Road";
  loadCapacity: number; // volume utilization %
  carbonMetric: number;  // gCO2e per ton-km
}

export interface DisruptionEvent {
  id: string;
  name: string;
  type: "natural" | "routing" | "bottleneck";
  latitude: number;
  longitude: number;
  radius: number; // Affected coverage in kilometers
  severity: number; // multiplier from 0 to 1; 1 is total closure
}

export interface SemanticEntity {
  id: string;
  name: string;
  type: "facility" | "route" | "agent" | "document";
  noteAssociations: string[]; // List of note filenames containing this entity
}
6.2 The Visual Node Graph Implementation Core (VisualNodeGraph.tsx)
A production-ready Canvas layout rendering node coordinates on screen, computing bezier links with animated data streams, and tracking interactions.
code
TypeScript
import React, { useRef, useEffect, useState } from "react";
import { Agent, NodePosition, NodeEdge, PantoneStyle } from "../types";

interface VisualNodeGraphProps {
  agents: Agent[];
  activeAgentIndex: number;
  setActiveAgentIndex: (index: number) => void;
  isRunning: boolean;
  currentStyle: PantoneStyle;
}

export default function VisualNodeGraph({
  agents,
  activeAgentIndex,
  setActiveAgentIndex,
  isRunning,
  currentStyle
}: VisualNodeGraphProps) {
  const canvasRef = useRef<HTMLCanvasElement | null>(null);
  const containerRef = useRef<HTMLDivElement | null>(null);
  const [nodes, setNodes] = useState<NodePosition[]>([]);
  const [edges, setEdges] = useState<NodeEdge[]>([]);
  const [draggedNodeId, setDraggedNodeId] = useState<string | null>(null);
  const [dragOffset, setDragOffset] = useState({ x: 0, y: 0 });

  // Initialize visual node locations symmetrically based on container sizing
  useEffect(() => {
    if (!containerRef.current) return;
    const { clientWidth, clientHeight } = containerRef.current;
    const width = clientWidth || 800;
    const height = clientHeight || 400;

    const computedNodes: NodePosition[] = agents.map((agent, i) => {
      const stepX = width / (agents.length + 1);
      return {
        id: agent.id,
        x: stepX * (i + 1),
        y: height / 2 + (i % 2 === 0 ? -30 : 30),
        vx: 0,
        vy: 0,
        width: 140,
        height: 60,
        isDragging: false
      };
    });

    const computedEdges: NodeEdge[] = [];
    for (let i = 0; i < agents.length - 1; i++) {
      computedEdges.push({
        id: `edge-${agents[i].id}-${agents[i+1].id}`,
        source: agents[i].id,
        target: agents[i+1].id,
        isActive: false,
        pulseProgress: 0
      });
    }

    setNodes(computedNodes);
    setEdges(computedEdges);
  }, [agents]);

  // Visual Node Graph Simulation Loop: Runs continuously
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    let animationFrameId: number;

    const render = () => {
      // Clear with dark-opacity matrix trail values for vector feel
      ctx.fillStyle = currentStyle.darkBg === "#000000" ? "rgba(0,0,0,0.85)" : "rgba(30,30,40,0.85)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // 1. Render Links (Bezier Curves with flows)
      edges.forEach((edge) => {
        const sourceNode = nodes.find((n) => n.id === edge.source);
        const targetNode = nodes.find((n) => n.id === edge.target);

        if (sourceNode && targetNode) {
          const x1 = sourceNode.x + sourceNode.width / 2;
          const y1 = sourceNode.y + sourceNode.height / 2;
          const x2 = targetNode.x - targetNode.width / 2;
          const y2 = targetNode.y + targetNode.height / 2;

          // Compute smooth control coordinates for bezier geometry
          const cpX1 = x1 + (x2 - x1) / 2;
          const cpY1 = y1;
          const cpX2 = x1 + (x2 - x1) / 2;
          const cpY2 = y2;

          ctx.beginPath();
          ctx.moveTo(x1, y1);
          ctx.bezierCurveTo(cpX1, cpY1, cpX2, cpY2, x2, y2);
          ctx.strokeStyle = isRunning ? currentStyle.primary : "rgba(100,116,139,0.3)";
          ctx.lineWidth = 2;
          ctx.stroke();

          // Live visual progress pulse tracking
          if (isRunning) {
            edge.pulseProgress += 0.01;
            if (edge.pulseProgress > 1) edge.pulseProgress = 0;

            const t = edge.pulseProgress;
            // Solve Bezier equation for localized point positioning
            const px = (1-t)**3 * x1 + 3*(1-t)**2*t * cpX1 + 3*(1-t)*t**2 * cpX2 + t**3 * x2;
            const py = (1-t)**3 * y1 + 3*(1-t)**2*t * cpY1 + 3*(1-t)*t**2 * cpY2 + t**3 * y2;

            ctx.beginPath();
            ctx.arc(px, py, 6, 0, Math.PI * 2);
            ctx.fillStyle = currentStyle.accent;
            ctx.shadowBlur = 10;
            ctx.shadowColor = currentStyle.accent;
            ctx.fill();
            ctx.shadowBlur = 0; // Reset canvas shadow state
          }
        }
      });

      // 2. Render Agent Nodes
      nodes.forEach((node, idx) => {
        const isCurrent = idx === activeAgentIndex;
        const agent = agents.find((a) => a.id === node.id);
        if (!agent) return;

        // Node Container Outer Fill & Gimmick styling
        ctx.fillStyle = isCurrent ? "rgba(30, 41, 59, 0.9)" : "rgba(15, 23, 42, 0.7)";
        ctx.strokeStyle = isCurrent ? currentStyle.primary : "rgba(100,116,139,0.5)";
        ctx.lineWidth = isCurrent ? 2 : 1;

        // Render card with round rect borders
        ctx.beginPath();
        const rx = node.x - node.width / 2;
        const ry = node.y - node.height / 2;
        ctx.roundRect(rx, ry, node.width, node.height, 8);
        ctx.fill();
        ctx.stroke();

        // Node Title Text
        ctx.font = "bold 13px 'Space Grotesk', system-ui";
        ctx.fillStyle = "#FFFFFF";
        ctx.fillText(agent.name.substring(0, 15), rx + 12, ry + 22);

        // Node Role Details
        ctx.font = "10px 'JetBrains Mono', monospace";
        ctx.fillStyle = isCurrent ? currentStyle.accent : "#94A3B8";
        ctx.fillText(agent.role.substring(0, 20) || "Agent Mode", rx + 12, ry + 38);

        // Parameter Badge tags
        ctx.font = "8px 'Inter', sans-serif";
        ctx.fillStyle = "rgba(255,255,255,0.4)";
        ctx.fillText(`T:${agent.temperature} | M:${agent.defaultModel.split("-")[0]}`, rx + 12, ry + 50);

        // Render anchor pins
        ctx.beginPath();
        ctx.arc(rx, ry + node.height / 2, 4, 0, Math.PI * 2);
        ctx.arc(rx + node.width, ry + node.height / 2, 4, 0, Math.PI * 2);
        ctx.fillStyle = currentStyle.primary;
        ctx.fill();
      });

      animationFrameId = requestAnimationFrame(render);
    };

    render();

    return () => {
      cancelAnimationFrame(animationFrameId);
    };
  }, [nodes, edges, isRunning, activeAgentIndex, currentStyle, agents]);

  // Handling Resize Events
  useEffect(() => {
    const handleResize = () => {
      const canvas = canvasRef.current;
      const container = containerRef.current;
      if (!canvas || !container) return;
      canvas.width = container.clientWidth;
      canvas.height = container.clientHeight || 400;
    };

    window.addEventListener("resize", handleResize);
    handleResize();

    return () => window.removeEventListener("resize", handleResize);
  }, []);

  // Mouse Interaction Callbacks
  const handleMouseDown = (e: React.MouseEvent<HTMLCanvasElement>) => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const rect = canvas.getBoundingClientRect();
    const mouseX = e.clientX - rect.left;
    const mouseY = e.clientY - rect.top;

    // Detect click intersections with nodes mapped back-to-front
    for (let i = nodes.length - 1; i >= 0; i--) {
      const node = nodes[i];
      const rx = node.x - node.width / 2;
      const ry = node.y - node.height / 2;
      if (
        mouseX >= rx &&
        mouseX <= rx + node.width &&
        mouseY >= ry &&
        mouseY <= ry + node.height
      ) {
        setDraggedNodeId(node.id);
        setActiveAgentIndex(i);
        setDragOffset({ x: mouseX - node.x, y: mouseY - node.y });
        break;
      }
    }
  };

  const handleMouseMove = (e: React.MouseEvent<HTMLCanvasElement>) => {
    if (!draggedNodeId) return;
    const canvas = canvasRef.current;
    if (!canvas) return;
    const rect = canvas.getBoundingClientRect();
    const mouseX = e.clientX - rect.left;
    const mouseY = e.clientY - rect.top;

    setNodes((prevNodes) =>
      prevNodes.map((node) =>
        node.id === draggedNodeId
          ? { ...node, x: mouseX - dragOffset.x, y: mouseY - dragOffset.y }
          : node
      )
    );
  };

  const handleMouseUp = () => {
    setDraggedNodeId(null);
  };

  return (
    <div ref={containerRef} className="w-full h-full relative min-h-[400px]" id="node-graph-container">
      <div className="absolute top-4 left-4 bg-slate-900/90 border border-slate-700 p-2.5 rounded-lg z-10 font-mono text-xs text-white">
        <p className="font-semibold text-slate-300">WOW Visual Node Orchestrator</p>
        <p className="text-slate-500 text-[10px]">Drag nodes to re-position. Dynamic execution visualizes stream direction.</p>
      </div>

      <canvas
        ref={canvasRef}
        onMouseDown={handleMouseDown}
        onMouseMove={handleMouseMove}
        onMouseUp={handleMouseUp}
        onMouseLeave={handleMouseUp}
        className="w-full h-full block cursor-grab active:cursor-grabbing border border-slate-700/50 rounded-xl"
        id="canvas-orchestrator-core"
      />
    </div>
  );
}
6.3 Digital Twin Forecast Engine Schema Processing
The simulation forecasting code executes server-side on server.ts. It takes spatial disruption signals, maps them geographically, simulates bottlenecks, and uses structured Gemini outputs to select remediation routes.
Below is the conceptual blueprint of the server handler parsing disruption schemas:
code
TypeScript
// server.ts snippet - Server-Side Digital Twin Predictive Routines
import { GoogleGenAI, Type } from "@google/genai";
import express from "express";

const router = express.Router();
const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

interface SimulatedImpactRequest {
  scenarioName: string;
  latitude: number;
  longitude: number;
  radiusKm: number;
  severity: number;
  gisFacilities: any[];
  purchaseContracts: any[];
}

router.post("/api/twin/forecast", async (req, res) => {
  try {
    const {
      scenarioName,
      latitude,
      longitude,
      radiusKm,
      severity,
      gisFacilities,
      purchaseContracts
    } = req.body as SimulatedImpactRequest;

    if (!process.env.GEMINI_API_KEY) {
      return res.status(500).json({ error: "GEMINI_API_KEY environment variable is required" });
    }

    // Step 1: Compute mathematical spatial impact overlaps (Haversine Formula)
    const earthRadiusKm = 6371;
    const hitFacilities = gisFacilities.map((facility) => {
      const lat1 = (latitude * Math.PI) / 180;
      const lat2 = (facility.latitude * Math.PI) / 180;
      const dLat = ((facility.latitude - latitude) * Math.PI) / 180;
      const dLon = ((facility.longitude - longitude) * Math.PI) / 180;

      const a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(lat1) * Math.cos(lat2) *
        Math.sin(dLon / 2) * Math.sin(dLon / 2);

      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
      const distance = earthRadiusKm * c;

      const isAffected = distance <= radiusKm;
      const dynamicThroughputLoss = isAffected ? facility.throughput * severity : 0;

      return {
        ...facility,
        distanceFromEpicenter: distance,
        isAffected,
        predictedThroughput: Math.max(0, facility.throughput - dynamicThroughputLoss)
      };
    });

    // Step 2: Feed affected components into Gemini structured reasoning schema
    const targetedPrompt = `
      You are an expert global logistics Digital Twin system assessing catastrophic risks.
      We have simulated the following disruption: "${scenarioName}" located near coordinates (${latitude}, ${longitude}) with active impact radius ${radiusKm}km.
      
      Below are the physical facility status reports detailing predicted operational capacities:
      ${JSON.stringify(hitFacilities, null, 2)}
      
      Evaluate the active purchase orders downstream of bottleneck terminals:
      ${JSON.stringify(purchaseContracts.slice(0, 15), null, 2)}
      
      Determine optimal remediation routing, alternative supplier zones, transport alternatives, and overall costs.
    `;

    // Define strict return schema for execution engine
    const responseSchema = {
      type: Type.OBJECT,
      properties: {
        disruptionAssessment: { type: Type.STRING, description: "Executive summary detailing structural supply impact" },
        bottleneckScore: { type: Type.NUMBER, description: "Risk rank from 0.00 (none) to 1.00 (catastrophic)" },
        reroutingPlan: {
          type: Type.OBJECT,
          properties: {
            suggestedAlternateRoutes: {
              type: Type.ARRAY,
              items: { type: Type.STRING },
              description: "Alternative sea, air, or rail corridors to bypass the epicenter"
            },
            carbonSurcharges: { type: Type.NUMBER, description: "Predicted overall delta cost index changes multiplier" }
          },
          required: ["suggestedAlternateRoutes", "carbonSurcharges"]
        }
      },
      required: ["disruptionAssessment", "bottleneckScore", "reroutingPlan"]
    };

    const response = await ai.models.generateContent({
      model: "gemini-3.1-flash",
      contents: targetedPrompt,
      config: {
        responseMimeType: "application/json",
        responseSchema: responseSchema,
        temperature: 0.2
      }
    });

    const parsedAssessment = JSON.parse(response.text);

    return res.json({
      status: "success",
      affectedFacilitiesList: hitFacilities,
      aiAnalysis: parsedAssessment
    });

  } catch (error: any) {
    console.error("Twin simulation forecasting failed", error);
    return res.status(500).json({ error: error.message || "Internal Forecasting Failure" });
  }
});
6.4 Note Keeper Semantic Graph Storage & RAG Engine
The RAG system stores unstructured markdown note indices in a responsive global state map. Below is the implementation blueprint for the client-side parsing routine that builds index edges.
code
TypeScript
// src/lib/semanticMemory.ts
import { GISRecord, Agent, SemanticEntity } from "../types";

export function rebuildSemanticIndex(
  notes: { filename: string; content: string }[],
  gisFacilities: GISRecord[],
  agents: Agent[]
): SemanticEntity[] {
  const entityRegister: Record<string, SemanticEntity> = {};

  // Register GIS Facility identifiers in matching register
  gisFacilities.forEach((facility) => {
    entityRegister[facility.facilityId.toLowerCase()] = {
      id: facility.facilityId,
      name: facility.facilityName,
      type: "facility",
      noteAssociations: []
    };
  });

  // Register Agent identifiers
  agents.forEach((agent) => {
    entityRegister[agent.id.toLowerCase()] = {
      id: agent.id,
      name: agent.name,
      type: "agent",
      noteAssociations: []
    };
  });

  // Parse markdown content line-by-line looking for hits
  notes.forEach((note) => {
    const rawNormalizedText = note.content.toLowerCase();

    Object.keys(entityRegister).forEach((key) => {
      // Find full-word boundaries to avoid false matching substrings
      const EscapedKey = key.replace(/[-\/\\^$*+?.()|[\]{}]/g, "\\$&");
      const pattern = new RegExp(`\\b${EscapedKey}\\b`, "g");

      if (pattern.test(rawNormalizedText)) {
        if (!entityRegister[key].noteAssociations.includes(note.filename)) {
          entityRegister[key].noteAssociations.push(note.filename);
        }
      }
    });
  });

  // Render arrays of valid entities back to layout components
  return Object.values(entityRegister).filter(
    (entity) => entity.noteAssociations.length > 0
  );
}
7. WORKSPACE THEMING, ERGONOMICS, AND PANTONE MATRIX INTEGRATION
The visual identity of this workspace is driven by responsive theme selection matching PANTONE color spaces, utilizing rich color combinations rather than boring default CSS palettes.
Theme Name	Primary Key	Secondary Key	Accent Color	Vibe / Purpose
Formula Red	#EF4444 (Vibrant Red)	#1E293B (Slate)	#F59E0B (Amber)	High intensity, alert tracking, simulation dashboards.
Classic Navy	#1E3A8A (Deep Blue)	#3B82F6 (Electric Blue)	#10B981 (Emerald)	Corporate oversight, secure audit flows, structured databases.
Forest Sage	#059669 (Rich Teal)	#022C22 (Emerald Green)	#A3E635 (Lime)	Sustainable distribution planning, carbon modeling, GIS.
Cyber Amber	#D97706 (Amber Gold)	#171717 (Neutral Dark)	#F59E0B (Yellow)	Terminal-like agent runner views, high-impact alerts.
code
CSS
/* index.css integration hooks */
@import "tailwindcss";
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Space+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

@theme {
  --font-sans: "Inter", "Space Grotesk", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "JetBrains Mono", ui-monospace, SFMono-Regular, monospace;
}

body {
  font-family: var(--font-sans);
  overflow-x: hidden;
}

/* Glassmorphic layout panels */
.workspace-glass-pane {
  background: var(--glass-bg);
  backdrop-filter: blur(12px);
  border: 1px solid var(--glass-border);
}

.workspace-pulse-glow {
  animation: visualGlowPulse 2.5s infinite ease-in-out;
}

@keyframes visualGlowPulse {
  0%, 100% {
    box-shadow: 0 0 5px var(--theme-primary-alpha-20);
    border-color: rgba(255,255,255,0.1);
  }
  50% {
    box-shadow: 0 0 20px var(--theme-primary-alpha-50);
    border-color: var(--theme-primary);
  }
}
8. BACKEND INTEGRATION, CLIENT CORS, AND SANDBOX ROUTING
Because this platform operates in a secure container sandbox routing files on port 3000 through Nginx, local calls are always relative to the active deployment:
code
JavaScript
// Secure endpoint fetching pattern in components
const endpointUrl = `/api/twin/forecast`;
try {
  const response = await fetch(endpointUrl, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ ...payload })
  });
  if (!response.ok) {
    const errorBody = await response.json();
    throw new Error(errorBody.error || "Execution failed");
  }
  const result = await response.json();
  // Map downstream changes directly back to React State variables
} catch (err) {
  // Clear graceful UI notification alerts without disrupting background thread cycles
}
9. 20 COMPREHENSIVE FOLLOW-UP QUESTIONS
To align this technical setup with your future deployment targets, please review these 20 development questions:
Architecture & Tech-Stack Integration
Model Versioning and Routing Balance: The current code references gemini-3.1-flash-lite for high-speed terminal chains. For advanced Digital Twin spatial reasoning logs, would you prefer routing these specifically to the full gemini-3.1-flash or the reasoning-optimized gemini-3.5-pro model to calculate more robust alternative routing vectors?
Streaming Protocol Architecture: During multi-agent runs, would you prefer to keep our client-side JSON polling loops, or upgrade the Node server pipeline to support Server-Sent Events (SSE) or WebSockets?
Database Scaling Strategy: The current structure loads initial datasets as in-memory client state. If your mock operational tables scale beyond 10,000 active records, should we integrate a persistent Firestore Database via set_up_firebase?
Local Vector Embeddings vs. Server API: For Note Keeper RAG searches, should we use our lightweight substring frequency matching algorithm, or run server-side vector calculations via Gemini's semantic text-embeddings API?
Multi-Key Customization Overrides: If external keys (OpenAI, Anthropic, xAI) are not registered on the settings screen, should the server-side proxy route automatically fall back to Gemini wrappers to prevent terminal process faults?
Workspace Visuals & User Experience
Canvas Layout Persistence: In the proposed Visual Agent Node Graph, should node coordinate edits be saved locally to localStorage so custom pipelines maintain their visual structure across page refreshes?
Simulation Disruption Visual Overlay: On the interactive GIS map, should shipping disruptions be represented as simple shaded circles or complex styled vector geometries (e.g., historical hurricane paths, actual port traffic congestion maps)?
RAG Link Actions: When a user clicks a highlighted inline semantic entity inside the Note Keeper, should the sidebar display a popover summary, or should the app split-screen and highlight the corresponding node directly in the GIS Tab?
Dual-Note Collaborative Integration: Would you like to introduce real-time collaboration widgets, or support side-by-side note comparison structures?
Pantone Custom Palette Injection: Would you like to allow operators to define custom Hex color values in JSON and save them as brand-new Pantone styles directly on the settings page?
Supply Chain Logistics Calculations
Disruption Severity Metrics: For the Monte Carlo calculations in the Digital Twin simulation, should impact severity vary dynamically based on transit mode (e.g., severe weather closing regional airports instantly, but only delaying ocean carriers by 48 hours)?
Carbon Offset Logic: In simulated throughput evaluations, should alternative transport suggestions prioritize reducing freight transit fees or minimizing overall CO₂ metrics?
Custom CSV Column Mapping: To support custom CSV files, should we build an interactive columns mapper dialog inside the Datasets tab, or require files to match our template column configurations?
Audit Compliance Scoring: Should the audit calculation engine verify balance sheets statically, or run dynamically (flagging issues like shipments departing before suppliers confirm receipt)?
Visual Trend Line Customization: In the Recharts execution metrics page, would you like to overlay predictive trend lines (like the ARIMA forecasting model) on top of active shipment timelines?
Production Lifecycle & Workspace Management
Token Consumption Controls: Should we add user-configurable limits (per user session/minute) to agent pipelines to safeguard developer billing budgets?
File Export and Note Packaging: When saving workspace notes, should the system package them as structured ZIP files along with spatial CSV records and operational PNG maps of the active node graph?
Multi-Modal Input Streams: For the Note Keeper, should we support voice recordings or hand-drawn maps (parsed via Gemini multimodal streams) to auto-generate markdown text templates?
Authorization and Access Controls: Should we restrict settings edits and API configuration changes behind a supervisor password role?
Self-Healing Agent Pipelines: If an agent node in the pipeline fails to respond, should the orchestrator automatically re-prompt the node or route the task to a designated fallback agent?
This technical specification details the complete architecture needed to build, configure, and refine a resilient next-generation multi-agent workspace. By wrapping your custom agent configuration pipelines in real-time node flows, predictive digital twin engines, and semantic memory notes, the workspace elevates raw operational metrics into actionable supply chain intelligence.
