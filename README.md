# Founder's Weekly Operating System — n8n Agentic Workflow

A multi-agent n8n workflow that turns a founder's scattered weekly inputs (meeting notes, customer feedback, Slack/email, a task list) into one **ranked, risk-aware executive plan**, gated by **human approval**.

This is a **strategic planning system**, not a chatbot: four AI agents with distinct roles hand off to deterministic logic for scoring, routing, and validation, with a founder-in-the-loop before anything is finalised.

---

## 1. Problem statement

| | |
|---|---|
| **User** | A startup founder / solo operator. |
| **Pain point** | Work is scattered across Slack, email, Notion, meetings, and customer feedback. The founder can't see *what matters most*, *what should be delegated*, or *what is silently blocking growth*. |
| **Why it matters** | Founders have limited hours; spending them on the wrong things (or missing a recurring problem that's quietly causing churn) is the most expensive mistake a startup makes. |
| **Workflow goal** | Collect the week's raw signals → prioritise objectively → surface systemic bottlenecks → produce a clear executive plan → require founder approval. |
| **Output** | A structured JSON plan: `top_3_priorities`, `biggest_risk`, `quick_wins`, `delegate`, `summary` — finalised only after the founder approves. |

---

## 2. Architecture

```
                                                    ┌─ Agent 2: PRIORITY (AI) ─→ Score & Rank (det.) ─┐
 Form Trigger ─→ Agent 1: COLLECTOR (AI) ─→ Parse & Validate (det.) ─→ Valid? (IF) ─┤                                                  ├─→ Merge (det.)
   (input)                                                       │ false              └─ Agent 3: BOTTLENECK (AI) ─→ Flag Systemic (det.)┘        │
                                                                 ▼                                                                              ▼
                                                       Fallback (bad input)                                          Assemble CEO Briefing (det.)
                                                                                                                                  │
                                                                                                                                  ▼
   Approved Plan (det.) ◄─ true ─┐                                                                                       Agent 4: CEO (AI)
                                 │                                                                                                │
   Rejected (det.) ◄── false ── Approved? (IF) ◄── Wait for Founder Approval (HUMAN) ◄── Validate Plan (det.) ◄────────────────────┘

   One shared Google Gemini model powers all four agents.
```

(`det.` = deterministic node. See the exported canvas / screenshots for the visual version.)

---

## 3. Node-by-node walkthrough

| # | Node | Type | AI or Deterministic | Responsibility |
|---|------|------|---------------------|----------------|
| 1 | **Founder Inputs** | Form Trigger | ⚙️ Deterministic | Collects the 4 raw input fields. *Where Slack/Gmail/Notion would plug in.* |
| 2 | **Agent 1 – Collector** | LLM Chain | 🤖 AI | Extracts messy text into structured JSON: `tasks`, `feedback_items`, `issues`. |
| 3 | **Parse & Validate** | Code | ⚙️ Deterministic | Parses Agent 1's JSON (strips code fences, `try/catch`). Sets `valid` flag. |
| 4 | **Valid Extraction?** | IF | ⚙️ Deterministic | Routes valid data forward; bad/empty input → fallback. |
| 5 | **Agent 2 – Priority** | LLM Chain | 🤖 AI | Rates each task on `urgency`, `revenue_impact`, `customer_impact` (1–5) + rationale. **Does not** compute the final score. |
| 6 | **Score & Rank** | Code | ⚙️ Deterministic | Weighted composite `= 0.40·urgency + 0.35·revenue + 0.25·customer`; sorts; buckets into Critical / Important / Backlog by threshold. |
| 7 | **Agent 3 – Bottleneck** | LLM Chain | 🤖 AI | Clusters recurring feedback/issues and counts frequency of each theme. |
| 8 | **Flag Bottlenecks** | Code | ⚙️ Deterministic | Marks any theme with `frequency ≥ 3` as `systemic`. |
| 9 | **Merge Agent Outputs** | Merge | ⚙️ Deterministic | Combines the two parallel branches (ranked tasks + bottlenecks). |
| 10 | **Assemble CEO Briefing** | Code | ⚙️ Deterministic | Builds one clean context object for the CEO agent. |
| 11 | **Agent 4 – CEO** | LLM Chain | 🤖 AI | Synthesises the final executive plan as structured JSON. |
| 12 | **Validate Plan** | Code | ⚙️ Deterministic | Confirms the plan is complete; formats a human-readable summary for review. |
| 13 | **Wait for Founder Approval** | Wait (form) | 👤 Human-in-the-loop | Pauses and shows the plan with **Approve / Reject** (no external account needed). |
| 14 | **Approved?** | IF | ⚙️ Deterministic | Branches on the founder's decision. |
| 15 | **Approved Plan** | Set | ⚙️ Deterministic | Finalises and timestamps the approved plan. |
| 16 | **Rejected** | Set | ⚙️ Deterministic | Logs the rejection + comments. |
| 17 | **Fallback – Bad Input** | Set | ⚙️ Deterministic | Friendly error path when extraction fails. |
| 18 | **Gemini Chat Model** | LLM sub-node | 🤖 AI (engine) | One free Gemini model shared by all four agents. |

---

## 4. Agentic practices demonstrated (mapped to the rubric)

- **Role definition / task decomposition** — four agents, each with a single, narrow responsibility (Collector, Priority, Bottleneck, CEO).
- **Structured outputs** — every agent is constrained to emit JSON with a fixed schema; downstream nodes depend on that contract.
- **AI vs deterministic split** — AI *judges* (extract, rate, cluster, synthesise); n8n *computes* (the scoring math, sorting, thresholds, routing, validation). The Priority agent deliberately does **not** score — that's deterministic, so rankings are consistent and explainable.
- **Routing / branching** — IF nodes for input validity and for the approval decision; a parallel split so Priority and Bottleneck agents run independently, then a Merge.
- **Human-in-the-loop** — the founder must approve the plan before it's finalised (high-impact output).
- **Validation & fallback** — JSON parsing with `try/catch`, a validity gate, and a dedicated fallback branch for unusable input.
- **Tool use** — Form input node (with documented hooks for Slack/Gmail/Notion/Sheets).

---

## 5. How to run

See **`SETUP.md`** for step-by-step setup (n8n install, free Gemini API key, import, run). In short:
1. Import `Founders_Weekly_OS.workflow.json` into n8n.
2. Add a free Google Gemini credential to the **Gemini Chat Model** node.
3. Run the workflow, fill the form with the data in **`sample_input.md`**.
4. Review the generated plan and click **Approve**.

A representative result is in **`sample_output.json`**.

---

## 6. Design decisions (why it's built this way)

- **Mocked sample data via a form**, not live integrations — keeps the demo 100% reproducible on camera, while the architecture clearly shows where real sources attach.
- **Scoring is deterministic, not AI** — an LLM asked to "rank tasks" is inconsistent run-to-run. Letting the AI rate dimensions but doing the weighted math in code makes priorities reproducible and auditable. This is the core AI-vs-deterministic decision.
- **Two agents in parallel** — Priority and Bottleneck are independent analyses, so they run concurrently and merge, mirroring how separate specialists would work.
- **Human approval before finalising** — a weekly plan is high-impact; a founder should sign off, so the workflow gates on it rather than auto-acting.
- **One shared LLM** — keeps cost/config trivial (one free Gemini key) while still presenting four distinct agent *roles* via their prompts.

---

## 7. Limitations & future improvements

- **Single LLM, no memory** — agents don't remember previous weeks; adding a datastore (e.g. a "last week's plan" lookup) would enable trend detection ("this bottleneck is now 3 weeks old").
- **Frequency counting is heuristic** — the Bottleneck agent estimates counts; a deterministic embedding/clustering step would make it exact.
- **Live integrations are mocked** — production would swap the form for Slack, Gmail, and Notion trigger/read nodes (and log approved plans to Google Sheets or Notion).
- **No retry loop on rejection** — a rejected plan currently just logs; a future version could feed the founder's comments back into the CEO agent to regenerate.

---

## 8. Repository contents

| File | What it is |
|------|-----------|
| `Founders_Weekly_OS.workflow.json` | The importable n8n workflow. |
| `README.md` | This file — problem statement + workflow explanation. |
| `SETUP.md` | Step-by-step setup and run guide. |
| `sample_input.md` | Realistic demo data to paste into the form. |
| `sample_output.json` | Representative final approved plan. |
| `screenshots/` | (You add) canvas + run screenshots — see the checklist in `SETUP.md`. |

> **Submission type:** Individual. The entire 4-agent workflow was designed and built as a single individual submission, so no group contribution note is required.
