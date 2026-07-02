# Mindsweeper — System Documentation (Master Index)

**Mindsweeper** is an agentic AI assessment platform for mathematics learners (Year 7+, including IB). It builds a rich, longitudinal profile of a student's *mathematical character* across multiple sessions, from three input channels: live Socratic dialogue, portfolio analysis, and validated psychological instruments.

This index links every component document. Each doc explains **what the component is, what it does, and how it connects to the others.**

---

## The system at a glance

```
                    ┌─────────────────────────────────────────────┐
                    │              STUDENT-FACING                  │
                    │                                              │
   ┌────────────────┴─────────────┐         ┌────────────────────┐ │
   │   ASSESSMENT AGENT (.jsx)     │ writes  │  STUDENT CONSOLE   │ │
   │  mindsweeper_agent_0625.jsx   ├────────►│ Mindsweeper_       │ │
   │                               │ session │ Student.html       │ │
   │  • intake → portfolio →       │  record └────────────────────┘ │
   │    M³ instruments → cognition │              ▲                  │
   │    task → Socratic/guided →   │              │ reads            │
   │    report                     │   localStorage (shared origin)  │
   └───────────────┬───────────────┘              │                  │
                   │ writes session record         │                  │
                   ▼                                │                  │
        ┌──────────────────────┐            ┌──────┴───────────────┐  │
        │  localStorage         │  reads     │  TEACHER CONSOLE     │  │
        │  CONTRACT             │◄───────────┤  Mindsweeper_Teacher │  │
        │  mindsweeper:index    │            │  _Console_1.html     │  │
        │  mindsweeper:student: │            │                      │  │
        └──────────────────────┘            │  • M³ trajectory      │  │
                                            │  • gaps over time     │  │
                                            │  • Capacity trend     │  │
                                            │  • session timeline   │  │
                                            └──────────────────────┘  │
                    └─────────────────────────────────────────────────┘
```

The agent **produces** a structured report per session and **appends** it to a per-student history in the browser's `localStorage`. Both consoles **read** that history (same-origin requirement). One session = one report; the consoles show development *across* sessions.

---

## Document set

| # | Document | What it covers |
|---|---|---|
| **00** | `00_MASTER_INDEX.md` | This file. System overview + index. |
| **01** | `01_AGENT_SYSTEM_PROMPTS.md` | The assessment agent's brain: the base Socratic protocol, the three dialogue modes (Socratic / Guided Question Walkthrough / Structured), the probe library, and the session phase flow. |
| **02** | `02_REPORT_SCHEMA.md` | The report JSON contract the agent emits — every field, every layer, all four schema variants (live / portfolio-only / transcript / template). |
| **03** | `03_VALIDATED_INSTRUMENTS.md` | The psychometric layer: SRQ-A (Motivation), MAI (Metacognition), CYRM-R (Resilience) — exact items, scoring formulas, banding. |
| **04** | `04_CAPACITY_LAYER.md` | The Cognition layer (`cognitive_profile`): why working memory **cannot** be read from chat, the validated span-task design (Digit Span + Listening Span, PCU scoring), and the build decisions it forces. |
| **05** | `05_CONSOLES.md` | The three rendering surfaces: Teacher Console, Student Console, and the in-agent report. What each shows and for whom. |
| **06** | `06_DATA_CONTRACT.md` | The `localStorage` contract that links agent → consoles: keys, record shape, append/dedup logic, deployment constraints. |
| **07** | `07_ARCHITECTURE_AND_DEPLOYMENT.md` | How the pieces fit, file manifest, deployment steps, the same-origin requirement, and known constraints (no backend, per-device storage). |
| **08** | `08_PERSONA_TEST_SUITE.md` | The five test personas and how they validate each diagnostic axis. |
| **agents/** | `agents/00_AGENT_SYSTEM_INDEX.md` + 8 files | The **eight-agent blackboard spec** — Orchestrator, the 6 assessors/adapters, and the **new Cognitive agent** (Capacity layer). Each agent's identity, inputs, owned schema fields, and methods. |

---

## The three-layer assessment model (the spine of everything)

Every observation Mindsweeper makes falls into exactly one of three layers. They are measured differently and never substitute for one another. This taxonomy drives the report schema, the agent's probes, and the console displays.

| Layer | Question it answers | Report home | How it's measured |
|---|---|---|---|
| **Content** (Knowledge) | What does the student *know*? Which schemas, facts, procedures are in place? | `knowledge_map` | Dialogue + portfolio evidence |
| **Capacity** (Cognition) | What does the maths *cost them in the moment*? How many steps can they hold; do they overload; do they offload? | `cognitive_profile` | **Span tasks (measured)** + chat load-signals (ecological) — see doc 04 |
| **Control** (Self-regulation / M³) | How do they *govern* the above? Mindset, motivation, metacognitive monitoring? | `mathematical_character` | Validated instruments + dialogue (convergence) |

**Key coupling:** Capacity and Control are linked, not identical. A student who writes the givens down before computing (a *Control* strategy) measurably *lowers* their *Capacity* load. Surfacing that relationship is one of the most actionable things a teacher gets.

---

## Methodological principle: triangulation

Every layer pairs a **validated/measured source** with a **behavioural observation**, and reports where they converge or diverge:

- **Control/M³** → instrument score (SRQ-A/MAI/CYRM-R) + how the student actually behaved in dialogue.
- **Capacity** → span-task score (measured WM) + ecological load signals from real maths under load.
- **Content** → portfolio evidence + live reasoning.

Convergence = a solid read. Divergence is *itself* the insight (e.g. self-reports high metacognition but can't articulate a strategy live; or strong digit span but collapses under processing load).

---

## Current build status

- **Built and working:** agent (all phases, three modes), report schema (incl. `cognitive_profile`), M³ instruments + scoring, all three consoles, the localStorage bridge, report rendering + HTML/PDF export.
- **Designed, not yet built:** the Capacity span-task widget + its scoring + the `verbal_stm_span` / `verbal_wm_span` schema fields (see doc 04). Until built, the chat produces *load signals only*, clearly labelled as inference, not measured capacity.
- **Deferred by design:** `ocean_signals` (Big Five from short dialogue is psychometrically weak), `context_factors` / relational resilience, and a cross-device backend.
