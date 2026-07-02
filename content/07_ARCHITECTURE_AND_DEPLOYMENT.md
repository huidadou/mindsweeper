# 07 · Architecture & Deployment

> How the pieces fit, the file manifest, deployment steps, and known constraints.

---

## Current architecture (the prototype)

Everything runs **client-side** in the browser. No server.

```
┌──────────────────────────────────────────────────────────────────┐
│  Static site (e.g. GitHub Pages, single origin)                   │
│                                                                    │
│   index.html ──(password gate)──► mindsweeper_agent_0625.jsx       │
│        │                              │                            │
│        │                              │ calls api.anthropic.com    │
│        │                              │ (claude-sonnet-4-6)        │
│        │                              ▼                            │
│        │                          report JSON                      │
│        │                              │                            │
│        │                              ▼ pushSessionToConsole()     │
│        │                       ┌──────────────────┐                │
│        │                       │  localStorage     │                │
│        │                       │  (shared origin)  │                │
│        │                       └──────┬───────────┘                │
│        │              reads           │           reads            │
│        ▼              ┌───────────────┴───────────────┐            │
│  (entry point)        ▼                               ▼            │
│             Mindsweeper_Teacher_      Mindsweeper_Student.html      │
│             Console_1.html                                         │
└──────────────────────────────────────────────────────────────────┘
```

- The agent calls the Anthropic API directly from the browser (a key is needed; see security note).
- The report is written to `localStorage`, appended per-student.
- Both consoles read that history (same-origin).

---

## File manifest (the live system)

| File | Role | Doc |
|---|---|---|
| `mindsweeper_agent_0625.jsx` | The assessment agent (all session logic, report generation, exports, the localStorage write side). | 01, 02 |
| `Mindsweeper_Teacher_Console_1.html` | Teacher longitudinal console (read side). | 05 |
| `Mindsweeper_Student.html` | Student-facing console (read side). | 05 |
| `index.html` | Deployed entry point (StatiCrypt password gate; serves the agent). | — |
| `CNAME` | Custom-domain config for GitHub Pages. | — |

### Supporting / source docs (in-repo, not runtime)
| File | Role |
|---|---|
| `state_schema_M3_lean.md` | **Single source of truth** for the report schema contract. |
| `SRQ-A_Motivation_16item.md`, `MAI_19item_HarrisonVallin.md`, `CYRM-R_Personal_Resilience.md` | The validated instruments (exact items + scoring). |
| `mindsweeper_regulation_probe_protocol.md` | Probe protocol detail. |
| `Mindsweeper_MCP_Server_Design.md` | Scoping doc for the planned MCP-server architecture (see below). |
| `Mindsweeper_M3_design_dialogue.md`, `mindsweeper_conversation_record.md` | Design history. |
| `README.md` | Persona test-suite guide (see doc 08). |
| `1..5_*_portfolio.md` / `1..5_*_transcript.md` | Five test personas' inputs. |
| `Mindsweeper_Demo_Brief_for_Team.md`, `SESSION_run_sheet.md` | Demo/run orchestration. |

---

## Deployment

1. Put the three runtime files **in the same folder** on a single-origin static host (GitHub Pages with the `CNAME`):
   - `mindsweeper_agent_0625.jsx` (served via `index.html`)
   - `Mindsweeper_Teacher_Console_1.html`
   - `Mindsweeper_Student.html`
2. Same origin is **required** — it's what lets the three share `localStorage` (doc 06).
3. The agent's console URLs are configurable constants (`TEACHER_CONSOLE_URL`, `STUDENT_CONSOLE_URL`); leave as `./Filename.html` when co-located.

---

## Security note — the API key

The agent calls the Anthropic API from the browser, which requires a key. **A live key must never be committed or shared in a deployed file** — anyone who can read the page can read the key and run up charges.

- The current file has `API_KEY = ""` (blank) so it falls back to the in-artifact proxy where applicable.
- For real browser deployment that calls the API, a key has to be present **and that is inherently exposed** — which is one of the main reasons the MCP-server migration (below) exists: it moves the key server-side.
- A key that was ever present in an uploaded file should be **rotated/revoked** at console.anthropic.com, regardless of later removal.

---

## Known constraints (current prototype)

| Constraint | Consequence | Resolution |
|---|---|---|
| No backend | Data is per-device, per-browser; no cross-device sync | MCP server / backend (planned) |
| `localStorage` in sandbox | Cross-session flow can't be tested in the Claude.ai preview | Test on the deployed site |
| Browser API key | Key is exposed in any deployed copy that calls the API | MCP server (key server-side) |
| Span tasks not built | Capacity is ecological inference only, not measured | Build the span-task widget (doc 04) |

---

## External integrations

| Integration | Used by | Purpose | Status |
|---|---|---|---|
| **Anthropic API** (`claude-sonnet-4-6`) | Agent | Runs the assessment dialogue + report generation | Live |
| **Learning Commons Knowledge Graph** (CASE + SAP Coherence Map) | Intervention Planner (Agent 8) | Authoritative standards, prerequisite/forward progressions, and Learning Components — the Content-layer sequencing backbone | Available as a connected tool (MCP) |

The Learning Commons Knowledge Graph exposes three tools the Intervention Planner calls: `find_standard_statement` (resolve a code → CASE UUID), `find_standards_progression_from_standard` (backward = prerequisites / unfinished learning, forward = next steps, per the Student Achievement Partners Coherence Map), and `find_learning_components_from_standard` (break a standard into teachable sub-skills). See `agents/AGENT_8_Intervention_Planner.md`.

---

## Planned: Mindsweeper MCP Server (designed, not built)

`Mindsweeper_MCP_Server_Design.md` scopes turning the browser prototype into a deployable **MCP server** any MCP client (Qoderwork, Claude.ai, Claude Code, ChatGPT, custom agents) can call. It solves the prototype's structural limits:

- **Logic server-side** → callable by other agent platforms (channel partnerships).
- **API key server-side** → safe for multi-tenant.
- **Persistence** → sessions are no longer isolated (enables true cross-device longitudinal history — the thing `localStorage` can't do).
- **Usage tracking** → billable.

Planned tool registry: `start_assessment_session`, `send_student_message`, `analyse_portfolio`, `analyse_transcript`, `generate_intervention_plan`, `get_session_report`. The report schema (doc 02) is the contract `get_session_report` returns, so the schema is forward-compatible with this migration.
