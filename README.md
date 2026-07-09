# Garden Assistance Agent Skill

A plug-and-play AI agent skill designed to turn LLMs (such as ChatGPT, Claude, Antigravity, or custom agent interfaces) into expert gardening assistants. 

This repository provides a complete, deterministic execution tool that enables AI agents to manage raised-bed crops, calculate soil-moisture water recommendations, and track scheduling calendars dynamically.

---

## Agent-Tool Integration Architecture

Rather than requiring users to run CLI commands directly, this skill separates the cognitive reasoning layer from the execution layer:

```text
  User (Natural Language) 
          │
          ▼
   AI Agent (LLM) ───[Reads SKILL.md]───► Maps intent to CLI arguments
          │
          ▼ (Executes Tool Call)
   scripts/garden_agent.py (Deterministic Engine) ───► Updates JSON Data
          │
          ▼ (Returns structured JSON)
   AI Agent (LLM) ───► Translates result to plain conversational advice
```

### Key Skill Features
* **Hallucination-Free Computations**: LLMs are notoriously poor at date arithmetic and multi-layer soil hydrology equations. This CLI acts as a deterministic sandbox tool that handles the math, leaving the LLM to focus on user experience.
* **Hemisphere-Agnostic Sowing calendars**: The engine shifts sowing windows dynamically relative to local frost boundaries (LFD/FFD) using Open-Meteo API lookups, meaning the agent can serve gardeners globally.
* **Stateless Piping Stream**: Fully supports piping via standard input/output streams (`--from-file -`), allowing conversational bots to invoke the skill without saving temporary files to disk.

---

## Workspace Structure

```text
garden-assistance/
├── SKILL.md                               # System instructions and intent maps loaded by LLMs
├── README.md                              # This file (Agent architecture documentation)
├── scripts/
│   └── garden_agent.py                    # The core CLI and simulation engine called by the agent
└── data/
    └── crop_knowledge_base.example.json   # Blank skeleton database template
```

---

## Conversational UX & Agent Tool-Calling

Here is how the AI agent maps conversational user prompts to deterministic tool calls under the hood:

### 1. Garden Initialization & Onboarding
* **User**: *"I am starting a garden in Los Angeles. Help me set up!"*
* **Agent Tool Call**:
  ```bash
  python3 scripts/garden_agent.py configure-profile \
    --latitude 34.0522 \
    --longitude -118.2437 \
    --timezone America/Los_Angeles \
    --bed-width 1.2 \
    --bed-length 3.0
  ```
  *Note: The CLI automatically fetches 365 days of historical weather data to extract local frost boundaries, self-initializing the garden profile.*

### 2. Sowing & Planting
* **User**: *"I sowed some carrots directly in the ground today."*
* **Agent Tool Call**:
  ```bash
  python3 scripts/garden_agent.py add-planted-crop \
    --plant-id carrot \
    --method outdoor_direct \
    --sown-date 2026-05-12
  ```

### 3. Smart Watering Inquiries
* **User**: *"Does the bed need water today?"*
* **Agent Tool Call**:
  ```bash
  python3 scripts/garden_agent.py watering-week --json
  ```
  *The CLI runs the FAO-56 dual crop coefficient equations against real-time weather forecasts and returns recommended volumes. The agent relays this as conversational advice (e.g., "Yes, apply 10 Liters...").*

### 4. Checklist Management
* **User**: *"Show me what garden chores are due."*
* **Agent Tool Call**:
  ```bash
  python3 scripts/garden_agent.py list-reminders --json
  ```

---

## License

This project is open-source software. You are free to copy, modify, and distribute it under your choice of licenses.
