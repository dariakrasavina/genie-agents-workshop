# Genie Agents Workshop — Facilitator Plan (GMF 2‑hour version)

A 2‑hour, hands‑on workshop that teaches attendees to **build, curate, evaluate, and operate a production‑ready Databricks Genie Agent** on manufacturing quality data — and to know when *not* to use one. This document is the facilitator's plan and a distilled best‑practices reference; the runnable content lives in `notebooks/`.

> **GMF version.** GMF is an independent GM subsidiary with its own environment rules. This branch (`gmf_workshop_version`) keeps notebooks **00–06 and 08**. It drops 03b (metric views), 07 (Genie Code skills), 09 (security), 10 (monitoring), 11 (CI/CD), and 12 (cleanup), and builds **two** agents (Baseline and Knowledge Store) instead of three. Participants share the **`containerunitycatalog`** catalog, and each person gets a schema named after their login (e.g. `ashish_arun`). The full 4‑hour version is on `main`.

> Terminology: **Genie Spaces** were renamed **Genie Agents**. This repo uses "Genie Agent" in prose; code identifiers, REST paths (`/api/2.0/genie/spaces`), and the `serialized_space` field keep their original names.

---

## 1. Goals & core message

The point of the session is **not** "add tables to Genie and watch it do magic." It's that a Genie Agent is only as good as the **data foundation and curation** behind it. Drive these messages home:

- **Data foundations first.** Genie is only as good as the gold‑layer, well‑modeled data that powers it (star schema / dimensional modeling; a governed semantic layer via **metric views**). This is a *prerequisite*, not the focus — mention it firmly, but don't turn the session into a data‑modeling class.
- **Fewer, better, trusted agents.** Sprawl kills adoption: a large fleet of thin, unvalidated agents sees very low real usage — at GM specifically, **2000+ agents were built but under ~2–3% are well‑adopted.** The goal is a small number of **high‑quality, benchmarked, certified** agents that users trust for decisions, ready to serve as building blocks in **multi‑agent orchestration** (e.g., GM's Glean).
- **Move the logic to the left.** Do as much as possible in the **data/semantic layer** (well‑modeled gold tables, metric views, SME‑defined calculations) rather than asking Genie to compute on the fly. Example: a battery/cell‑engineering agent where a chemistry‑engineer SME's domain calculations were built *into the data layer* instead of prompted at query time.
- **Curation is the work.** Most low‑adoption agents skipped the parts that matter: **measures, filters, fields/dimensions, joins, synonyms, well‑chosen SQL examples, focused instructions, and benchmarks.** That maturity is what earns trust.
- **Trust is measurable.** Replace "it seems to work" with **benchmarks** and monitoring. Aim for **80%+ benchmark accuracy before user acceptance testing**, then keep validating in production.

## 2. Audience & format

- **~100 attendees, mixed:** data/technical **builders** plus **business consumers**; some new to Databricks. Design for two tracks in one room:
  - **Builders** — create and curate the agent (the hands‑on core).
  - **Consumers** — learn to *use* the agent and to feed quality signals back (thumbs up/down, request‑review, suggested questions). Their feedback is the reinforcement loop that improves agents.
- **2 hours, no scheduled break** (GMF session; date TBD). Presenter‑led (one presenter) with **DSAs as TAs** for Q&A; no GM‑side presentations (packed week). Optional: a GM SME introduces "here's how this applies to our teams" live.
- **Hands‑on is the priority.** Everyone runs the **same notebooks together** so results are consistent; also **show the UI** so click‑oriented users see both paths.
- **Domain SMEs matter.** The most valuable agents come from builders collaborating with subject‑matter experts who supply domain logic — and from pushing calculations **into the data/semantic layer** rather than asking Genie to compute them on the fly.

## 3. When to use a Genie Agent vs. a BI dashboard (say this early)

| Use case | Reach for | Why |
|---|---|---|
| **Deterministic, recurring metrics** you monitor daily (e.g., units produced, first‑pass yield, safety incidents, a KPI wall) | **AI/BI Dashboard** (optionally on metric views) | You want to see many numbers at once, the same way every time — low cognitive load, no re‑asking. |
| **Exploratory / conversational / anomaly follow‑up** ("market share dropped — why?", "which lines drove scrap last month?") | **Genie Agent** | Flexible aggregation and grouping; ad‑hoc questions that a fixed dashboard can't anticipate. |
| **One‑off question about a single table** | UC sample‑data tab / a quick query | Not worth a curated agent. |

Genie **augments** dashboards; it doesn't replace them. Don't push users to re‑ask the same deterministic question repeatedly — that doesn't scale.

## 4. Agenda (2 hours) — `H` hands‑on · `T` talk · `D` demo

| Time | Segment | Fmt | Notebook / asset |
|---|---|---|---|
| 0:00–0:10 | Orient · when to use a Genie Agent vs. a BI dashboard · **curation earns trust** message (incl. metric views as a concept, no hands‑on) | T | slides |
| 0:10–0:25 | **Config + clean data:** check catalog/schema, load 7 tables, comments + functions | H | `00`, `01`, `02` |
| 0:25–0:40 | **Build your baseline agent** — programmatically **and** in the UI; try the canonical questions | H | `03` |
| 0:40–1:10 | **Knowledge Store:** synonyms, joins, measures, filters, fields, example SQL, minimal instructions | H | `04` |
| 1:10–1:30 | **Evaluate with Benchmarks:** push to both agents, run in the UI, accept snippets / update ground truth | H | `05` |
| 1:30–1:45 | **Ask your own questions** in the Knowledge Store agent; consumer feedback (thumbs / request review) | H | `06` |
| 1:45–1:55 | **Proof:** Baseline vs. Knowledge Store on 4 unseen questions | D/H | `08` |
| 1:55–2:00 | Wrap‑up · what's next · Q&A · delete your agents | T | — |

**If you're running behind:** skip the optional "Update instructions programmatically" cell in `05`. In `06`, have attendees ask two or three questions instead of the full list. If needed, the presenter can run `08` as a demo instead of everyone running it.

---

## 5. Best‑practices reference (the knowledge to teach)

### 5.1 Product taxonomy
- **Genie Agents** — curated, domain‑specific environments where data teams define trusted data, metrics, and business rules. (Formerly "Genie Spaces.")
- **Genie Code** — for builders: generate/run code, pipelines, dashboards, debug in notebooks/SQL/MLflow.
- **Genie One** — the business‑user surface to ask questions and explore across workspaces (now also in **Slack/Teams** and a **mobile app**, Public Preview).

### 5.2 Start with clean, well‑curated data
- **Focused datasets:** start with a small set of UC tables (**~3–7**, max 30 per agent); remove unnecessary/conflicting columns. Focused, non‑conflicting data is the single biggest driver of accurate SQL.
- **Curated / gold data:** prefer gold‑level tables or pre‑aggregated views to avoid complex joins and heavy calculations.
- **UC metadata is the most effective context:** table & column **comments** (describe the grain and example values), **PK/FK** relationships (or pre‑joined views).

### 5.3 Tables vs. metric views
- **Metric views** — pre‑define metrics, dimensions, and aggregations in Unity Catalog: governed, reusable, **deterministic by design**, with flexible grouping. Best when KPIs must be trustworthy. Trade‑off: you author/maintain the metric‑view definition, and Genie is limited to the view's perimeter.
- **Tables** — simplest to start and best for **exploratory** questions across complex, linked data; but require good descriptions/PK‑FK and risk Genie choosing the wrong formula for undefined KPIs.
- **Guidance:** use metric views to consolidate related tables (and stay within the 30‑table limit) and to lock down business‑critical KPIs; use tables for open exploration.

### 5.4 The Knowledge Store (where teams under‑invest)
Scoped to the agent (doesn't change Unity Catalog metadata). Component types:

- **Local data prep:** hide/exclude columns (no new view needed), edit table/column **descriptions** locally, add column **synonyms** (Genie‑specific natural‑language references — cryptic column names → business terms).
- **Joins:** uses UC PK/FK; define joins **locally** when you can't set keys; Genie can suggest joins from past queries/notebooks. Relationship types: many‑to‑one, one‑to‑many, one‑to‑one, many‑to‑many.
- **SQL expressions** — reusable definitions for standard business concepts. Three flavors:
  - **Measures** — KPI calculations, so Genie doesn't guess the math (e.g., avg OEE, scrap rate, first‑pass yield, YoY growth).
  - **Filters** — common conditions / global filters (e.g., exclude test data).
  - **Fields / dimensions** — row‑level attributes to slice, categorize, or reformat (e.g., `CASE WHEN … END`, `CONCAT`, `SUBSTRING`, `DATEDIFF`, `COALESCE`; mileage buckets, repair‑order types).
- **Example SQL queries** — a natural‑language question paired with its SQL, applied dynamically to guide Genie on similar prompts. Highest‑impact after metadata. Cover a spread of patterns (aggregation, 2‑ and 3‑table joins, ratios, date ranges, subquery/HAVING, MIN/MAX).
- **Prompt matching** — representative values (format assistance) and curated value lists (entity matching) for common filter columns.

**Priority order for teaching Genie logic:** SQL expressions (measures/filters/fields) → example SQL → **text instructions only as a last resort**. Keep it consistent — don't let text conflict with SQL.

### 5.5 Programmatic curation (`serialized_space`)
The Knowledge Store **can be built in code**, not only in the UI — via the `serialized_space` JSON on `POST`/`PATCH /api/2.0/genie/spaces`. Reconstructed shape (from the Genie REST API reference and the Genie Workbench source; **confirm the exact shape by `GET`‑ing a UI‑curated agent with `include_serialized_space=true` before relying on it**):

```
serialized_space = {
  "data_sources": {
    "tables":       [ { "identifier", "description",
                        "column_configs": [ {"column_name","description","synonyms","exclude","enable_matching"} ] } ],
    "metric_views": [ ... ]
  },
  "sample_questions": [ ... ],
  "text_instructions": [ ... ],
  "example_sqls":      [ {"question","sql","usage_guidance","parameters":[{"name","type_hint","description","default_value"}]} ],
  "measures":     [ {"display_name","sql","alias","synonyms","instruction","comment"} ],
  "filters":      [ {"display_name","sql","alias","synonyms","instruction","comment"} ],
  "expressions":  [ {"display_name","sql","alias","synonyms","instruction","comment"} ],
  "join_specs":   [ {"left_table","right_table","left_column","right_column","relationship","instruction","comment"} ],
  "benchmarks":   [ {"question","sql", ...} ]
}
```
> Note: the repo's current `serialized_space` payloads use an earlier v2 layout (`instructions.text_instructions`, `instructions.example_question_sqls`, `benchmarks.questions`, id‑keyed arrays‑of‑strings). Reconcile against a live dump before building the full Knowledge Store notebook. `synonyms` and `exclude`/hide‑columns appear settable via `column_configs`; verify.

### 5.6 Text instructions — do's & don'ts
**Do:** teach Genie **when to ask for clarification**; add **formatting** rules (number format, language, row limits); be **concise and directive** ("When the user… always…"); organize as a bulleted list.
**Don't:** overload text (it isn't prompt‑filtered — it eats context and invites conflicts); add **conflicting** instructions; enumerate column values (use value dictionaries / example values); put **SQL logic** in text (use example SQL and SQL expressions).
**Clarification pattern (4 parts):** (1) trigger condition → (2) what's missing → (3) require a clarifying question → (4) example question.
> *"When users ask about performance breakdown but don't include time range or channel, ask a clarification question first — e.g., 'Please specify the time range and channel.'"*

### 5.7 Benchmarks & evaluation
- **What:** curated test questions paired with **gold‑standard SQL**; evaluations compare Genie's results to the gold answer to measure data accuracy, catch hallucinations, and safely roll out.
- **Framework:** (1) define **10–20 realistic questions** with SMEs **before** development; (2) establish a **baseline**; (3) optimize **one change at a time**, re‑running the full suite; (4) measure & communicate. **Target 80%+ before UAT.**
- **Result semantics:** green = pass; **yellow = needs manual review** (scorer not confident); red = fail; **red warning ⊙ = could not evaluate** (missing gold SQL, or zero matching rows between output and ground truth).
- Re‑run after every change; keep a benchmark‑run history.

### 5.8 Monitoring, feedback & the improvement loop
- **Monitor tab / Weekly digest:** see every question & answer, filter by time/user/rating/status, step through conversation threads, review thumbs up/down and review requests.
- **Thumbs don't auto‑update the agent** — authors review signals first, then improve context.
- **Audit logs + alerts:** email notifications on actions like thumbs‑down or request‑review.
- **Consumer actions to teach:** rate answers, "request review" with a comment, suggest better questions — this is how non‑builders improve the agent.
- **Loop:** triage feedback → fix in order (**data/views → metadata → joins → example SQL → instructions**) → re‑run benchmarks → promote good interactions to curated examples → repeat.

### 5.9 Conversation API best practices (for embedding)
- **Queue requests** — the API doesn't manage retries.
- **Poll every 5–10s** until a terminal status (`COMPLETED` / `FAILED` / `CANCELLED`); cap polling at ~10 minutes.
- **Exponential backoff** if nothing after ~2 minutes.
- **Start a new conversation per session** — reusing threads across sessions reduces accuracy from unintended context.

### 5.10 Rollout & management
- **Self‑validation** → **staged rollout** (small trusted group, then expand) → **ongoing iteration**.
- **Certify** high‑quality agents with labels; retire the rest. Adoption follows trust, and trust follows curation + benchmarks.
- **Pair agents with dashboards** to orient users; "Top drivers" (GA) helps explain *why* a metric moved.

### 5.11 What's new / on the horizon (worth demoing or mentioning)
- **Embed a Genie Agent as an iframe** — now **GA**.
- **Conversation / Genie Agents API** for custom apps (see 5.9).
- **Genie One in Slack & Microsoft Teams** (`@mention`), and a **mobile app** — Public Preview.
- **Genie Workbench** — a Databricks App to create, IQ‑score, and auto‑optimize agents (worth showing; not a curation API).
- **Multi‑agent orchestration** — well‑curated, certified domain agents as building blocks.

---

## 6. GMF version — what changed vs. `main`

- **Kept:** `00`, `01`, `02`, `03`, `04`, `05`, `06`, `08`.
- **Removed:** `03b` metric views, `07` Genie Code skills (and `skill/`), `09` security, `10` monitoring, `11` CI/CD, `12` cleanup.
- **Two agents, not three:** `05` pushes a single 10‑question suite to the Baseline and Knowledge Store agents. The metric‑view suite and the 4‑question smoke test are gone, because `08` covers that comparison. `08` is now a two‑way compare.
- **GMF environment (from GMF's suggested corrections):** the default catalog is `containerunitycatalog` and the schema defaults to the login name. `01`/`02` backtick‑quote the schema in `CREATE SCHEMA` / `CREATE VOLUME` / `USE SCHEMA`.

## 7. Open items to confirm
- **Live `serialized_space` dump** from a UI‑curated agent — the `04` builder targets the Workbench‑confirmed shape; verify field-for-field against a real dump (`GET …?include_serialized_space=true`) before the session and reconcile if anything differs.
- **Confirm GMF schema naming:** check that each participant's schema in `containerunitycatalog` matches their login (e.g. `ashish_arun`), or tell attendees which name to put in the Schema widget.
- **Registration list pending** (from the coordinator) → confirms the audience mix; **finance** is the notable interested function. If one function dominates, add a tailored example; otherwise keep manufacturing.
- **Entitlements confirmed** for all registrants (sandbox + workspace + SQL + consumer) — the entitlements/platform team to verify against the list; see `PREREQUISITES.md`. Prework text has already been broadcast to registrants.
- Optional **SME live use‑case intro** slot ("here's how this applies to our teams").

---

## References
- Internal deck: *Best Practices for Curating and Managing Genie Agents.*
- Stakeholder planning call (discovery) — audience, format, and content priorities.
- Databricks docs: Genie Agents — overview, **concepts**, **set‑up**, **monitor**; **Curate an effective Genie Agent** (best practices); **Knowledge Store**; **Genie REST API reference**.
- Blog: *How to Build Production‑Ready Genie Spaces, and Build Trust Along the Way.*
- **Genie Workbench** — `github.com/databricks-solutions/databricks-genie-workbench`.
