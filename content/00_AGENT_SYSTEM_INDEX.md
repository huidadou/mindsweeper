# Mindsweeper Multi-Agent System — Architecture & Index

> The instruction set for Mindsweeper's **eight-agent blackboard architecture**.
> Each agent has its own instruction file in this folder. This index explains how they fit together.

---

## What this is (and an honest note on status)

Mindsweeper's *design* is a **blackboard multi-agent system**: specialised assessor agents observe a student, write behavioural primitives to a shared "blackboard," and an Orchestrator routes those signals into the report schema.

**Status:** the current shipping prototype (`mindsweeper_agent_0625.jsx`) implements all of this behaviour inside **one large system prompt** — it is not yet split into separate running agents. These files are the **specification** for the agent separation: they define each agent's identity, inputs, outputs, and methods so the system can be (a) reasoned about cleanly, (b) split into real agents (e.g. per-tool in the planned MCP server), and (c) extended — which is exactly what adding the Cognitive agent does.

The schema doc states the principle directly: *"M³ is a presentation/taxonomy layer only — the seven-agent blackboard does not mirror it 1:1. Assessors collect behavioural primitives; the Orchestrator routes their signals into these fields."*

---

## The blackboard model

```
        ┌──────────────────────────────────────────────────────────┐
        │                     THE BLACKBOARD                        │
        │  a shared, append-only store of behavioural primitives    │
        │  (observations, scores, evidence) keyed by source agent   │
        └──────────────────────────────────────────────────────────┘
              ▲          ▲          ▲          ▲          ▲
   writes     │          │          │          │          │
  ┌───────────┴┐ ┌───────┴────┐ ┌───┴──────┐ ┌─┴────────┐ ┌┴─────────────┐
  │ Knowledge   │ │ Mindset &  │ │ Cognitive│ │Metacog. &│ │ Resilience & │
  │ Mapper      │ │ Motivation │ │  (NEW)   │ │  Comm.   │ │  Affect      │
  │ (Content)   │ │ (Control)  │ │(Capacity)│ │ (Control)│ │ (Control)    │
  └─────────────┘ └────────────┘ └──────────┘ └──────────┘ └──────────────┘
         ▲                                                         
         │ feeds evidence (not an assessor)                        
  ┌──────┴───────┐                          reads blackboard, assembles report
  │ Portfolio    │                            ┌────────────────────────────┐
  │ Ingestion    │                            │       ORCHESTRATOR          │
  └──────────────┘                            │  routes signals → schema    │
                                              │  resolves convergence       │
                                              └──────────────┬─────────────┘
                                                             │ assembled report
                                                             ▼
                                              ┌────────────────────────────┐
                                              │   Intervention Planner       │
                                              │  consumes the report →       │
                                              │  recommendations             │
                                              └────────────────────────────┘
```

Assessors don't talk to each other — they only write to the blackboard. The Orchestrator is the only agent that reads everything and owns the final schema. This decoupling is why the report taxonomy (M³) doesn't have to mirror the agent roster.

---

## The eight agents

| # | Agent | Type | Layer | Owns (schema) | File |
|---|---|---|---|---|---|
| 1 | **Orchestrator** | Controller | — | assembles whole report; `convergence`, `overall_verdict`, `narrative_summary` | `AGENT_1_Orchestrator.md` |
| 2 | **Knowledge Mapper** | Assessor | Content | `knowledge_map` | `AGENT_2_Knowledge_Mapper.md` |
| 3 | **Mindset & Motivation** | Assessor | Control | `mathematical_character.mindset`, `.motivation` | `AGENT_3_Mindset_Motivation.md` |
| 4 | **Metacognition & Communication** | Assessor | Control | `mathematical_character.metacognition`, `.self_assessment_accuracy` | `AGENT_4_Metacognition_Communication.md` |
| 5 | **Resilience & Affect** | Assessor | Control | `mindset.subdimensions.resilient_disposition` (+ affect) | `AGENT_5_Resilience_Affect.md` |
| 6 | **Cognitive** ⭐ NEW | Assessor | **Capacity** | `cognitive_profile` (incl. `verbal_stm_span`, `verbal_wm_span`) | `AGENT_6_Cognitive.md` |
| 7 | **Portfolio Ingestion** | Evidence adapter | (feeds Content) | writes evidence + `requires_chat` flags; **not** an assessor | `AGENT_7_Portfolio_Ingestion.md` |
| 8 | **Intervention Planner** | Consumer | — | reads assembled report → intervention plan, grounded in named **science-of-learning** principles + the **Learning Commons Knowledge Graph** | `AGENT_8_Intervention_Planner.md` |

⭐ **The Cognitive agent is the new addition.** The three-layer model (Content / Capacity / Control) previously had assessors for Content and Control but **none for Capacity** — the cognitive-load signals were homeless. The Cognitive agent owns the Capacity layer: it administers the span tasks (measured WM) and reads in-session load signals (ecological), and writes both to `cognitive_profile`.

---

## The three-layer mapping (which agents serve which layer)

| Layer | Question | Agents | Schema |
|---|---|---|---|
| **Content** | What do they know? | Knowledge Mapper (+ Portfolio Ingestion as evidence) | `knowledge_map` |
| **Capacity** | What does it cost them in the moment? | **Cognitive** | `cognitive_profile` |
| **Control** | How do they govern it? | Mindset & Motivation; Metacognition & Communication; Resilience & Affect | `mathematical_character` |

The Orchestrator and Intervention Planner sit outside the layers — one assembles, one consumes.

---

## Shared conventions (all assessor agents follow these)

1. **Write primitives, not conclusions.** An assessor writes observations + evidence to the blackboard. The Orchestrator does the final classification and cross-agent reconciliation.
2. **Evidence is mandatory.** Every signal carries an exact quote or described behaviour. No evidence → don't write it.
3. **`requires_chat` / `not_observed`, never fabricate.** If a mode can't surface a signal (e.g. portfolio-only), say so explicitly.
4. **Triangulate.** Where an agent has both a validated/measured source and a behavioural observation, it writes both and lets the Orchestrator compute `convergence`. Divergence is diagnostic, not noise.
5. **No teaching, no answers, no empty praise** (assessors that talk to the student).
6. **Gender-neutral language** (they/them) throughout.

---

## Reading order

Start with **AGENT_1_Orchestrator** (it defines the blackboard contract the others write to), then the assessors (2–6), then the adapter (7) and consumer (8). The new Cognitive agent (6) is the one to read most closely if you're implementing the Capacity layer.
