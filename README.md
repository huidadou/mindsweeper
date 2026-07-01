<div align="center">

# 🧭 Mindsweeper

**An agentic AI system for assessing mathematical *character*, not just mathematical answers.**

*Not a test — a sweep.*

[![Website](https://img.shields.io/badge/website-mindsweeper.sg-0F6E56)](https://mindsweeper.sg)
[![Status](https://img.shields.io/badge/status-working%20prototype%20(TRL%204–5)-b87a15)](#technical-readiness)
[![Built for](https://img.shields.io/badge/Hackastone-AI%20for%20Education-1e40af)](#)

</div>

---

## What is Mindsweeper?

Most maths assessments find the wrong mine. They catch *what* a student got wrong, never *why* — and miss entirely what lies underneath: the fixed mindset that quits at the first wall, the exam anxiety that blanks under pressure, the working-memory limit mistaken for carelessness.

Mindsweeper runs **eight specialised AI agents** in parallel around a shared student model to produce, in a single **~30-minute session**, the kind of diagnosis that normally takes a human tutor weeks of observation. The output is not a score. It is a structured, evidence-cited profile of how a student **thinks, feels, and fights**.

It works across **11 curricula** — US Common Core/AP, IB MYP & DP (AA/AI, SL/HL), UK GCSE/IGCSE, UK A-Level, Singapore O-Level/IP, Singapore A-Level, and China Gaokao.

---

## The framework: Content · Capacity · Control

Every observation belongs to one of three layers, mirroring the cognitive architecture itself:

| Layer | Question it answers | Produced by |
|---|---|---|
| **Content** | *What does the student know?* | Knowledge map — mastery by strand, error taxonomy, transfer, readiness |
| **Capacity** | *What can they hold in mind right now?* | Cognitive profile — measured span tasks + in-session load |
| **Control** | *How do they govern the above?* | Mathematical character — **M³: Mindset · Motivation · Metacognition** + resilience |

**Control is grounded in validated psychometric instruments** (never paraphrased): the **SRQ-A** (motivation, Relative Autonomy Index), the **MAI** (metacognition; Harrison & Vallin 2018 short form), and the **CYRM-R** Personal subscale (resilience). **Capacity is measured**, not inferred — via a forward **digit-span** task and a **word-span** task, kept strictly separate from ecological load signals.

Mindsweeper deliberately does **not** infer personality traits (OCEAN/Big Five): single-session trait inference is psychometrically weak and ethically fraught.

---

## Architecture — eight agents, one blackboard

Specialised agents read from and write to a shared student model (the *blackboard*), so evidence from every source accumulates into one coherent profile.

| # | Agent | Layer | Role |
|---|---|---|---|
| 1 | **Orchestrator** | — | Sequences the assessment; selects each probe; assembles the report, convergence, and verdict |
| 2 | **Knowledge Mapper** | Content | Mastery, error taxonomy, transfer, root-gap fan-out |
| 3 | **Mindset & Motivation** | Control | SDT autonomy spectrum · growth vs fixed |
| 4 | **Metacognition & Communication** | Control | Calibration gap · ICAP explanation depth |
| 5 | **Resilience & Affect** | Control | Cliff vs slope · anxiety markers |
| 6 | **Cognitive** | Capacity | Span tasks + in-session load; convergence between them |
| 7 | **Portfolio Ingestion** | — | Structured reading of uploaded student work |
| 8 | **Intervention Planner** | — | Evidence-tagged plan; cites science-of-learning; consults the Learning Commons Knowledge Graph |

The live Socratic assessor runs a **perceive → decide → act → reflect** loop and emits a structured `<trace>` on every turn (probe deployed, signals detected, profile update, cognitive-load read) — making the reasoning inspectable rather than a black box.

**Two assessment modes:** *Open Socratic* (static assessment — a clean baseline) and *Choose-your-question* (dynamic assessment — the agent teaches *through* a chosen problem and measures learning potential; the level of support required is the score).

---

## Reliability

Because this is an *assessment* tool, output consistency is a first-order requirement — and LLMs are stochastic. Mindsweeper addresses this architecturally:

- **Deterministic foundation** — the validated instruments and span tasks are scored by code from the learner's own responses. Identical responses → identical scores. This never touches the model's stochasticity.
- **Constrained LLM variance** — every model call uses **`temperature: 0`**; every measurement-bearing field resolves to a **fixed anchor-value schema** (e.g. mindset ∈ {fixed-leaning, mixed, growth-leaning}), turning free description into scoreable classification; unrecognised values coerce to `null` and are never fabricated.
- **Validated at the construct level** — planned **test–retest** with **ICC** (continuous scores) and **Cohen's κ** (classifications).

See [`Mindsweeper_Reliability_Methodology.md`](./Mindsweeper_Reliability_Methodology.md) for the full write-up.

---

## Repository contents

### Deployable surfaces (served from one origin)

| File | What it is |
|---|---|
| [`index.html`](./index.html) | Project landing page |
| [`mindsweeper_app.html`](./mindsweeper_app.html) | **Live** assessment agent (asks for your own Anthropic API key, stored only in your browser) |
| [`mindsweeper_app_demo.html`](./mindsweeper_app_demo.html) | **Demo** agent — fully scripted Yu Xiao Ting walkthrough, no key needed |
| [`Mindsweeper_Teacher_Console_demo.html`](./Mindsweeper_Teacher_Console_demo.html) | Teacher console with a worked longitudinal example |
| [`Mindsweeper_Teacher_Console_blank.html`](./Mindsweeper_Teacher_Console_blank.html) | Empty-state teacher console for live use |
| `Mindsweeper_Teacher_Console_{mi,kai,aanya,jay,mei}.html` | Five persona demo dashboards |
| [`Mindsweeper_Flow.html`](./Mindsweeper_Flow.html) | Visual system flowchart |

### Documentation

| File | What it is |
|---|---|
| [`Mindsweeper_Submission_Report.md`](./Mindsweeper_Submission_Report.md) | Full competition submission report |
| [`Mindsweeper_Reliability_Methodology.md`](./Mindsweeper_Reliability_Methodology.md) | Reliability & reproducibility methodology |
| [`Mindsweeper_Question_Bank.md`](./Mindsweeper_Question_Bank.md) | Curated question bank by curriculum |

---

## Running it

**End users need nothing installed** — it's browser-only.

- **Try the demo:** open `mindsweeper_app_demo.html` (or visit the deployed site). Fully scripted, no API key.
- **Run a live assessment:** open `mindsweeper_app.html` and paste your own Anthropic API key when prompted. The key is stored **only** in your browser's `localStorage` and is never committed or sent anywhere except the Anthropic API.

**Same-origin note:** the app and the teacher console share data through a browser `localStorage` contract (`mindsweeper:index`, `mindsweeper:student:<id>`), so completed sessions appear in the console only when both files are served from the **same origin** (e.g. the same GitHub Pages site).

### Building the agent from source

The agent is authored in React/JSX (`mindsweeper*.jsx`) and bundled to a standalone HTML file with [esbuild](https://esbuild.github.io/). The two console surfaces are static HTML and need no build step.

```bash
npm install
node build_html2.mjs   # emits the inlined mindsweeper_app*.html bundles
```

The demo build sets `DEMO_MODE = true`; the live build sets it `false`.

---

## Technical readiness

Mindsweeper is at roughly **TRL 4–5**: a working prototype, demonstrated end-to-end. All surfaces run and the agentic assessment functions on real student cases.

The known gap is stated plainly: **there is no backend yet.** The `localStorage` contract is same-origin and single-device; there is no auth or per-user identity. Crucially, the hard part — the assessment methodology — is what's built; what remains is standard, well-understood engineering (database, auth, hosting).

**Next milestone (weeks, not months):** persistence (Postgres/Supabase) → authentication → server-side API proxy → unified hosting on mindsweeper.sg.

---

## Team

- **Zheng Jiahui** — Team Leader (MSc Science of Learning, NIE, NTU; and MSc Technopreneurship and Innovation (Chinese), NTU)
- **Gao Siqiao** — MSc Science of Learning (NIE, NTU)
- **Advisor:** Prof. Cai Yiyu (Nanyang Technological University)
- **Technical consultant:** Wang Quchao (National University of Singapore, MSc Data Science and Machine Learning, DSML)

Built for **Hackastone: Global Contest on AI for Education**.

---

## License & data

No student data is collected or transmitted by this repository. In the live app, the Anthropic API key you supply stays in your browser. Session data written by the demo lives in `sessionStorage` and is cleared when the tab closes.

<div align="center">

**[mindsweeper.sg](https://mindsweeper.sg)** · *Precision Learning — not a test, a sweep.*

</div>
