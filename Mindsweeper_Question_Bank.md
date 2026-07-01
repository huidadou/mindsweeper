# Mindsweeper — Question Bank

*The curated question set used in the Choose-your-question (dynamic assessment) mode.*

---

## Purpose and design

In **Choose-your-question** mode, the student is offered **three calibrated questions** — one **algebraic**, one **geometric**, and one **applied** — and picks the one they want to work through. The agent then teaches *through* that single question with a graduated prompting ladder, measuring learning potential.

Rather than generating these questions with a live model call at runtime, Mindsweeper serves them from an **instant, curated `QUESTION_BANK`** keyed by curriculum track. This is a deliberate reliability and cost decision:

- **Deterministic and instant** — no network round-trip, no generation latency, no risk of a malformed or off-level question reaching a student mid-assessment.
- **Level-appropriate by construction** — each set is hand-calibrated to the difficulty and notation conventions of its curriculum, so a Gaokao student sees Gaokao-style items and an IB AA HL student sees HL-level calculus.
- **One of each type** — every set provides exactly one algebraic, one geometric, and one applied question, so the three-way choice always spans the breadth a diagnosis needs. The student's *choice itself* is a signal (risk appetite, topic confidence).

Each entry carries a `topic`, a standards code where applicable (`ccss` — populated for US Common Core, "—" elsewhere), and a `type` tag (`algebraic` / `geometric` / `applied`).

> **Status — provisional.** This bank is a **temporary, working set** used to make the current prototype fully deterministic and demonstrable. It will be **expanded and refined further** — more items per curriculum, finer difficulty banding, richer portfolio-gap targeting, and broader topic coverage — as the product matures. Treat the questions below as representative of the design, not as a final or exhaustive list.

## Selection logic

When a session reaches the Choose-your-question step:

1. The student's **syllabus** (from intake) selects the matching curriculum set.
2. If the track is `Other / Not sure` or otherwise unrecognised, the bank falls back to the **UK GCSE / IGCSE** set (`__default`).
3. Portfolio findings are available to bias selection toward a flagged gap where appropriate; the default behaviour returns the curriculum's standard trio.

> **Demo note.** In `DEMO_MODE`, this step returns a fixed scripted trio tuned to the Yu Xiao Ting persona (a reverse-percentage problem, a perimeter/area quadratic system, and a linear-modelling tank problem) so the walkthrough is identical every time.

## Coverage

**11 curricula**, each with a full algebraic / geometric / applied trio, plus a default fallback:

US Common Core / AP · IB MYP · IB DP — AA SL · IB DP — AA HL · IB DP — AI SL · IB DP — AI HL · UK GCSE / IGCSE · UK A-Level · Singapore O-Level / IP · Singapore A-Level · China Gaokao 高考 · (`Other / Not sure` → UK GCSE / IGCSE fallback)

---

## The bank

### US Common Core / AP

| Type | Topic | Standard | Question |
|---|---|---|---|
| Algebraic | Quadratic equations | A.REI.B.4 | Solve the quadratic equation 2x² − 7x + 3 = 0 by any method. |
| Geometric | Circle & line | A.REI.C.7 | A circle has equation x² + y² = 25. Find the coordinates of the points where the line y = x + 1 meets it. |
| Applied | Quadratic modelling | F.IF.B.4 | A ball's height in metres after t seconds is h = 20t − 5t². When does it hit the ground, and what is its maximum height? |

### IB MYP

| Type | Topic | Question |
|---|---|---|
| Algebraic | Linear equations | Solve for x: 3(2x − 1) = 4x + 7. |
| Geometric | Pythagoras & area | A right-angled triangle has legs of 6 cm and 8 cm. Find the hypotenuse and the area of the triangle. |
| Applied | Proportional reasoning | A car travels 240 km in 3 hours. Travelling at the same rate, how long does it take to cover 400 km? |

### IB DP — AA SL

| Type | Topic | Question |
|---|---|---|
| Algebraic | Discriminant | The quadratic x² + kx + 9 = 0 has two equal roots. Find the possible values of k. |
| Geometric | Differentiation — tangents | Find the equation of the tangent to the curve y = x² − 4x + 5 at the point where x = 3. |
| Applied | Sequences & series | An arithmetic sequence has u₁ = 5 and u₁₀ = 32. Find the common difference and the sum of the first 10 terms. |

### IB DP — AA HL

| Type | Topic | Question |
|---|---|---|
| Algebraic | Logarithms | Solve for x: log₂(x) + log₂(x − 2) = 3. |
| Geometric | Integration — area between curves | Find the exact area of the region enclosed between the curves y = x² and y = 2x. |
| Applied | Kinematics | A particle's displacement is s = t³ − 6t² + 9t (metres, t in seconds). Find the times when the velocity is zero, and the total distance travelled in the first 3 seconds. |

### IB DP — AI SL

| Type | Topic | Question |
|---|---|---|
| Algebraic | Linear models | The cost of producing n items is C = 40 + 15n dollars. Find n when the cost is $250, and explain what the 40 represents. |
| Geometric | 3-D geometry | A cone has radius 5 cm and height 12 cm. Find its slant height, then its total surface area. |
| Applied | Exponential models | A town of 5000 people grows by 4% each year. Write a model for the population after n years, and find the population after 6 years. |

### IB DP — AI HL

| Type | Topic | Question |
|---|---|---|
| Algebraic | Exponential decay | A quantity decays according to A = 200·e^(−0.03t). Find the value of t for which A = 80. |
| Geometric | Trigonometry — area rule | Find the area of a triangle with two sides of 7 cm and 9 cm and an included angle of 52°. |
| Applied | Modelling — Newton cooling | A cup of coffee cools according to T = 20 + 60·e^(−0.05t) (°C, t in minutes). Find the temperature after 10 minutes, and how long until it reaches 40 °C. |

### UK GCSE / IGCSE *(also the default fallback)*

| Type | Topic | Question |
|---|---|---|
| Algebraic | Simultaneous equations | Solve the simultaneous equations: 3x + 2y = 16 and x − y = 2. |
| Geometric | Pythagoras & trigonometry | In a right-angled triangle the hypotenuse is 13 cm and one leg is 5 cm. Find the other leg, then the smallest angle in the triangle. |
| Applied | Percentages | A £60 jacket is reduced by 15% in a sale. Find the sale price. It is later raised back to £60 — what percentage increase is that? |

### UK A-Level

| Type | Topic | Question |
|---|---|---|
| Algebraic | Discriminant | Find the values of k for which x² + (k − 3)x + 4 = 0 has no real roots. |
| Geometric | Differentiation — stationary points | Find the coordinates of the turning point of y = x² − 6x + 11, and state whether it is a maximum or minimum. |
| Applied | Kinematics | A particle moves with velocity v = 3t² − 12t + 9 (m/s). Find the times when it is instantaneously at rest, and its acceleration at t = 1. |

### Singapore O-Level / IP

| Type | Topic | Question |
|---|---|---|
| Algebraic | Quadratic factorisation | Solve x² − 3x − 10 = 0 by factorisation. |
| Geometric | Pythagoras (applied) | A ladder 5 m long leans against a wall with its foot 1.4 m from the base of the wall. How high up the wall does the ladder reach? |
| Applied | Scale & ratio | A map has a scale of 1 : 50 000. Two towns are 8 cm apart on the map. Find the actual distance between them, in kilometres. |

### Singapore A-Level

| Type | Topic | Question |
|---|---|---|
| Algebraic | Quadratic inequalities | Find the range of values of x for which x² − 5x + 6 > 0. |
| Geometric | Differentiation — normals | Find the equation of the normal to the curve y = x² − 2x at the point (3, 3). |
| Applied | Exponential growth | A bacterial population is N = 500·e^(0.2t) (t in hours). Find how long it takes for the population to double. |

### China Gaokao 高考

| Type | Topic | Question |
|---|---|---|
| Algebraic | Quadratic inequalities | Given f(x) = x² − 2x − 3, find the set of values of x for which f(x) < 0. |
| Geometric | Line & circle — tangency | The line y = 2x + c is a tangent to the circle x² + y² = 5. Find the possible values of c. |
| Applied | Sequences & series | In an arithmetic sequence, a₃ = 7 and a₇ = 19. Find a₁, the common difference, and the sum of the first 10 terms. |

---

*Implemented as the `QUESTION_BANK` object in the agent source; `Other / Not sure` and any unrecognised syllabus resolve to the UK GCSE / IGCSE set via `QUESTION_BANK["__default"]`. This set is provisional and will continue to be updated.*
