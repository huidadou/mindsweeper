# Mindsweeper — Reliability Methodology

*How Mindsweeper produces consistent, defensible assessment results — and how we validate it.*

Version 0.5 · For reviewers and technical evaluators

---

## The question this document answers

> "If the same student is assessed twice, will Mindsweeper produce the same result? LLMs are stochastic — how do you guarantee reliability in an assessment context?"

This is the right question to ask of any assessment tool. Our answer is not "the LLM always outputs the same thing" — that would be neither true nor, for an adaptive assessment, desirable. Our answer is architectural: **the measurement-bearing parts of the system are deterministic and reproducible; the LLM is confined to probing and expression, with its variance actively constrained; and reliability is validated at the construct level using standard psychometric statistics.**

---

## 1. The core principle: separate measurement from expression

Reliability in psychometrics does not require *verbatim* identical output. It requires **construct-level consistency** — that the underlying judgement about a learner (is their motivation introjected or autonomous? is their resilience a cliff or a slope?) is stable across administrations. Whether the report phrases that judgement with slightly different wording is irrelevant to reliability.

Mindsweeper is therefore built so that every component is explicitly classified as either **deterministic** (reproducible by construction) or **LLM-mediated** (variance-constrained and validated statistically).

| Component | Produced by | Reproducibility |
|---|---|---|
| SRQ-A motivation scoring (RAI, four subscales) | Local scoring algorithm | **Deterministic** — identical responses → identical scores |
| MAI metacognition scoring (Knowledge / Regulation of Cognition) | Local scoring algorithm | **Deterministic** |
| CYRM-R resilience scoring (Personal subscale sum) | Local scoring algorithm | **Deterministic** |
| Digit-span / Word-span capacity | Local measurement logic | **Deterministic** |
| Error classification, knowledge-strand mastery | LLM + enforced schema | **High** — anchored categories, scoreable by κ |
| Character classification (mindset / motivation / resilience type) | LLM + enforced schema | **High** — anchored categories |
| Socratic dialogue path | LLM (adaptive) | **Low by design** — see §4 |
| Narrative summary (prose for teachers) | LLM (temperature 0) | Wording varies; construct judgement stable |

The **foundation of the assessment — the validated psychometric instruments and the span tasks — never touches the LLM's stochasticity.** SRQ-A, MAI, and CYRM-R are scored by deterministic code from the learner's own item responses. Two identical response sets yield byte-identical scores. This is the bedrock on which reliability rests.

---

## 2. What we do to constrain LLM variance

For the parts the LLM does mediate, we apply three layers of control:

### 2.1 Temperature = 0
All model calls (Socratic dialogue, portfolio analysis, portfolio report, transcript analysis) are issued with `temperature: 0`, i.e. greedy decoding. This removes sampling-induced variation and is the single largest reduction in output variance.

**Honest caveat:** `temperature = 0` is *near*-deterministic, not perfectly deterministic. Floating-point non-associativity in parallel GPU inference, batching, and hardware differences mean identical inputs can occasionally produce a different token. We do not claim bit-for-bit reproducibility from the model; we claim that sampling variance is eliminated and residual variance is small and non-systematic.

### 2.2 Enforced output schema (anchored classification)
Every **measurement-bearing** field resolves to a fixed set of anchor values — not free text. For example:

- `mindset.classification` ∈ { `fixed leaning`, `mixed`, `growth leaning` }
- `motivation.type` ∈ { `extrinsic`, `mixed`, `intrinsic` }
- `resilience.pattern` ∈ { `cliff`, `slope`, `plateau` }
- `knowledge_map.overall_level` ∈ { `below grade`, `at grade`, `above grade` }

This converts the LLM's task from *"describe this student"* (free description, high wording variance, unscoreable) to *"select the nearest anchor"* (categorical choice, low variance, directly scoreable with Cohen's κ).

Enforcement is implemented in two complementary ways:

1. **Coercion layer (live today):** parsed output is passed through a normaliser that snaps every measurement field to its nearest legal anchor — handling case, hyphenation, and synonym drift (e.g. "Growth-leaning", "growth mindset", "growth leaning (confident)" all resolve to `growth leaning`). If no legal value is present, the field is set to `null` — the system never fabricates a category that was not observed.
2. **Schema-forced tool call (contract defined):** the report schema is also expressed as a JSON-Schema tool definition (`emit_report`). Issued as a dedicated call with `tool_choice: { type: "tool" }`, the API *guarantees* the output conforms — correct field names, enum values, and types — with no regex extraction. This is the stricter enforcement path and is wired as a one-call refactor when report generation is separated from the final conversational turn.

Free-text fields that are **expression, not measurement** (the teacher-facing narrative summary, evidence quotes) remain prose. They are allowed to vary in wording; they carry no scored value.

### 2.3 Fixed rubrics
Each classification is made against explicit, versioned rubric criteria in the agent instructions (e.g. what behavioural evidence licenses a `cliff` vs `slope` resilience pattern), and every classification must cite the specific in-session moment that evidences it. This anchors the LLM's judgement to observable evidence rather than impression.

---

## 3. Reliability validation plan (test–retest)

Constraining variance is necessary but not sufficient; reliability must be *measured*. We adopt the standard psychometric design: **test–retest** — administer to the same cohort twice, separated by a short interval (target: two weeks, short enough that the underlying constructs are stable, long enough to avoid simple recall), and quantify agreement.

**Sample:** target ≥ 50 learners, Year 7 and above, spread across curricula (aligned with the roadmap's validation-study milestone).

**Statistics:**

- **Continuous scores** (SRQ-A RAI, MAI KoC/RoC, CYRM-R sum): **Intraclass Correlation Coefficient (ICC)**.
  - Interpretation: ICC > 0.70 acceptable, > 0.80 good, > 0.90 excellent.
  - Expected: because these are deterministically scored from item responses, test–retest ICC for the instruments approaches the ceiling and reflects genuine trait stability rather than tool noise — this is a strength to report explicitly.
- **Categorical classifications** (mindset, motivation type, resilience level/pattern, error categories): **Cohen's κ**.
  - Interpretation: κ 0.61–0.80 substantial, > 0.80 almost perfect.
  - This is where the anchored-schema work pays off: because outputs are categorical, agreement is directly computable.

**Additional checks:**
- **Inter-rater agreement** between Mindsweeper's classifications and blinded expert-teacher classifications on the same evidence (construct validity, not just reliability).
- **Internal consistency** of the deployed instrument subsets (Cronbach's α) to confirm the shortened SRQ-A / MAI / CYRM-R retain their factor structure.

---

## 4. Why the dialogue is *deliberately* not reproducible

The Socratic conversation is adaptive: the Orchestrator selects each next probe based on the learner's previous answer and which assessor currently has the highest uncertainty. If a learner answers differently, the conversation branches — by design.

This is not a reliability defect. It is the same principle as **computerised adaptive testing (CAT)**: two students (or the same student on two occasions) legitimately receive different item sequences, yet the *estimated construct* is what must be stable, not the item path. A fixed-script interview would be more reproducible and *less* valid. We optimise for the stability of the inferred profile, not the stability of the transcript.

---

## 5. Summary statement (for reviewers)

> The reliability of Mindsweeper rests on a deterministic foundation: the validated instruments (SRQ-A, MAI, CYRM-R) and the span tasks are scored by code and are fully reproducible. The LLM is confined to probing and expression; its variance is constrained with temperature 0, an enforced anchor-value schema, and fixed evidence-cited rubrics, so that measurement outputs are categorical and scoreable. We validate at the construct level via test–retest — ICC for continuous scores, Cohen's κ for classifications — rather than claiming verbatim identity, which for an adaptive assessment would be neither achievable nor appropriate.

---

*References: Koo & Li (2016) on ICC selection and interpretation; McHugh (2012) on Cohen's κ in reliability studies; Wainer (2000) on computerised adaptive testing; the instrument manuals for SRQ-A, MAI (Harrison & Vallin 2018), and CYRM-R for their published reliability properties.*
