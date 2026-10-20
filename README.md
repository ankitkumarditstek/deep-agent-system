# 🧠 Deep Agents Research Lab (`DeepAgentic`)

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Framework: LangGraph](https://img.shields.io/badge/orchestration-LangGraph-orange.svg)](https://github.com/langchain-ai/langgraph)
[![Framework: Deep Agents](https://img.shields.io/badge/agents-DeepAgents-purple.svg)](https://github.com/langchain-ai)
[![UI: Streamlit](https://img.shields.io/badge/interface-Streamlit-red.svg)](https://streamlit.io/)

**Deep Agents Research Lab** is an enterprise-grade multi-agent research and reasoning system built with LangChain, LangGraph, and Deep Agents. It combines hierarchical model-driven subagents, pluggable agent skills, filesystem-based persistent memory, grounded local evidence retrieval (SQLite + local document store), and strict Human-in-the-Loop (HITL) execution controls.

---

## 🌟 Key Features

- **Hierarchical Multi-Agent Architecture**:
  - **Orchestrator Agent (`research_agent`)**: Oversees inquiry resolution, selects operational skills, coordinates subagents, and generates verified structured research reports.
  - **`researcher` Subagent**: Queries local documents and relational SQLite records to extract raw evidence without inventing citations.
  - **`analyst` Subagent**: Synthesizes verified findings, performs comparative analysis, and distinguishes concrete facts from speculative interpretations.
- **🛡️ Human-in-the-Loop (HITL) Approvals**:
  - Dangerous or sensitive actions (e.g., file deletion, outbound communication) trigger LangGraph interrupts.
  - Review pending actions directly in the Streamlit UI and selectively approve or reject execution before the agent proceeds.
- **📚 Modular Skills & Persistent Memory**:
  - Pluggable modular skill guidelines (e.g., `research`, `finance` in `workspace/skills/`).
  - Cross-turn persistent agent persona and system guidelines tracked in `workspace/memory/AGENTS.md`.
- **📊 Verified Structured Output**:
  - Enforces schema consistency via Pydantic (`ResearchReport`), producing a verified summary, bulleted key findings, actual cited sources, and confidence ratings.
- **💻 Interactive Streamlit Interface**:
  - Live chat-based research interface with thread-based session memory.
  - Document evidence uploader (`.txt`, `.md`) with instant workspace indexing.
  - Interactive approval cards for pending action requests.

---

## 🏗️ Architecture Overview

```mermaid
graph TD
    User([User / Streamlit UI]) <--> Orchestrator[Orchestrator: research_agent]
    
    subgraph Multi-Agent Hierarchy
        Orchestrator -->|Delegate Search| Researcher[Researcher Subagent]
        Orchestrator -->|Delegate Analysis| Analyst[Analyst Subagent]
    end

    subgraph Knowledge & Tools
        Researcher --> SearchDocs[Tool: search_docs]
        Researcher --> SearchDB[Tool: search_database]
        Analyst --> SearchDB
        SearchDocs --> DocsDir[("workspace/docs (*.txt, *.md)")]
        SearchDB --> SQLiteDB[("workspace/company.db (SQLite)")]
    end

    subgraph Memory & Skills
        Orchestrator -.-> Skills[Skills: /skills/research, /skills/finance]
        Orchestrator -.-> Memory[Memory: /memory/AGENTS.md]
    end

    subgraph Human-in-the-Loop Safety Gate
        Orchestrator -->|Interrupt on sensitive tools| SafetyGate{Requires Approval?}
        SafetyGate -->|Approved| SendEmail[Tool: send_email (Outbox)]
        SafetyGate -->|Approved| DeleteFile[Tool: delete_file (Docs)]
        SafetyGate -.->|Approval Prompt| User
    end

    Orchestrator -->|Pydantic Structured Output| Report[ResearchReport: Summary, Findings, Sources, Confidence]
    Report --> User
```

---

## 📁 Project Structure

```plaintext
deep_agentic_system/
├── app.py                  # Streamlit web application & HITL interface
├── deep_agent.py           # Agent definition, subagents, memory & LangGraph config
├── setup_data.py           # Local docs and SQLite sample data initialisation
├── tools.py                # Agent tools: search_docs, search_database, send_email, delete_file
├── requirements.txt        # Python dependency manifest
├── .env.example            # Environment variable configuration template
└── workspace/              # Sandboxed agent runtime workspace
    ├── company.db          # Local SQLite knowledge database
    ├── docs/               # Indexed research documents (*.txt, *.md)
    ├── memory/
    │   └── AGENTS.md       # Long-term agent memory and style preferences
    ├── outbox/             # Safe local outbox for simulated emails
    └── skills/             # On-demand domain-specific agent skills
        ├── finance/
        │   └── SKILL.md    # Financial analysis directives
        └── research/
            └── SKILL.md    # Evidence gathering & citation directives
```

---

## 🚀 Getting Started

### 1. Prerequisites

- Python 3.10 or higher
- An active [OpenAI API key](https://platform.openai.com/api-keys)

### 2. Clone and Setup Environment

```bash
# Navigate to the project directory
cd "deep_agentic_system"

# Create a virtual environment
python -m venv venv

# Activate the virtual environment
# Windows (PowerShell):
.\venv\Scripts\Activate.ps1
# Linux / macOS:
source venv/bin/activate

# Install required dependencies
pip install -r requirements.txt
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```bash
cp .env.example .env
```

Edit `.env` and set your OpenAI API key:

```env
OPENAI_API_KEY=sk-proj-your-actual-api-key-here
OPENAI_MODEL=gpt-4.1-mini
```

### 4. Initialize Local Evidence & Knowledge Base

Run the setup script to seed `workspace/docs/` and create `workspace/company.db`:

```bash
python setup_data.py
```

### 5. Launch the Streamlit Web App

```bash
streamlit run app.py
```

Open your browser to `http://localhost:8501` to access the **Deep Agents Research Lab**.

---

## 🔍 How It Works

1. **Query Ingestion**: Enter a research prompt into the chat bar or click **"▶ Run sample research"**.
2. **Skill & Context Resolution**: The agent consults `workspace/memory/AGENTS.md` and appropriate skills in `workspace/skills/` to plan the investigation.
3. **Evidence Retrieval**:
   - The `researcher` subagent scans local files via `search_docs` and queries the SQLite database via `search_database`.
   - The `analyst` subagent cross-references records and formulates grounded insights.
4. **Safety Verification**:
   - If the agent attempts to write an email or delete a document, execution halts. An interactive approval prompt is presented in the UI to approve or reject the action.
5. **Report Generation**: Output is parsed into a structured report displaying the Executive Summary, Key Findings, Cited Sources, and Confidence score.

---

## 🛠️ Customization & Extensibility

- **Adding New Documents**: Drop `.txt` or `.md` files directly into `workspace/docs/` or use the file uploader in the Streamlit sidebar.
- **Adding New Skills**: Create a new folder inside `workspace/skills/<skill_name>/` containing a `SKILL.md` with YAML frontmatter specifying its name and triggers.
- **Custom Tools**: Define new `@tool` decorated functions in `tools.py` and register them in `deep_agent.py`.
- **Adjusting Guardrails**: Add or remove tools in the `interrupt_on={...}` mapping in `deep_agent.py` to change which actions require human authorization.

