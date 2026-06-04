# Setup & Run Guide

Everything you need to import, configure, and run the workflow — and capture the screenshots for your GitHub repo. The **only** thing that costs nothing-but-a-signup is the Gemini API key; every other node is credential-free.

---

## Step 1 — Get n8n running

Pick **one**:

**Option A — n8n Cloud (easiest, no install).**
1. Go to <https://n8n.io> → "Get started" → create a free trial account.
2. You'll land in the n8n editor in your browser.

**Option B — Run locally (free, needs Node.js 18+).**
```bash
npx n8n
```
Then open <http://localhost:5678> in your browser. (First run downloads n8n; give it a minute.)

---

## Step 2 — Get a free Google Gemini API key

1. Go to **Google AI Studio**: <https://aistudio.google.com/app/apikey>
2. Sign in with a Google account → **Create API key** (free tier, no credit card).
3. Copy the key.

> Want OpenAI instead? Swap the **Gemini Chat Model** node for an **OpenAI Chat Model** node, connect it to the four agents the same way, and use an OpenAI key. Gemini's free tier is recommended for this assignment.

---

## Step 3 — Import the workflow

1. In n8n, top-right menu (**⋯**) → **Import from File…**
2. Select `Founders_Weekly_OS.workflow.json`.
3. The canvas appears with ~18 nodes + sticky-note labels.

---

## Step 4 — Connect the Gemini credential

1. Double-click the **Gemini Chat Model** node (bottom of the canvas).
2. In **Credential for Google Gemini(PaLM) API** → **Create New Credential**.
3. Paste your API key from Step 2 → **Save**.
4. That single model node already feeds all four agents — you only set this once.

---

## Step 5 — Run it

1. Click **Open form** on the **Founder Inputs** node (or click **Execute Workflow** / **Test workflow** at the bottom — n8n gives you the form URL).
2. Paste the four blocks from **`sample_input.md`** into the matching form fields → **Submit**.
3. Watch the nodes light up green left-to-right. The two AI branches (Priority + Bottleneck) run in parallel.
4. When it reaches **Wait for Founder Approval**, n8n shows an approval form with the generated plan. Choose **Approve** (or Reject) → **Submit**.
5. The workflow finishes on **Approved Plan** (or **Rejected**). Click that node to see the final JSON output.

---

## Step 6 — Capture screenshots for the repo

Create a `screenshots/` folder and grab:

- [ ] `01-canvas.png` — the full workflow canvas (zoom to fit).
- [ ] `02-form.png` — the input form filled with sample data.
- [ ] `03-agent-output.png` — one agent node's structured JSON output (click a node → Output tab).
- [ ] `04-score-rank.png` — the Score & Rank node output showing composite scores + buckets.
- [ ] `05-approval-form.png` — the founder approval form showing the plan.
- [ ] `06-final-plan.png` — the Approved Plan node's final JSON.

Then push everything to a GitHub repo (workflow JSON, README, sample files, screenshots).

```bash
cd /path/to/multiagent
git init
git add .
git commit -m "Founder's Weekly OS — n8n agentic workflow"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| A node shows "credential not set" | Only the **Gemini Chat Model** needs a credential — set it (Step 4). |
| Import warns about a node version | Your n8n is older than a node's `typeVersion`. Open the node, n8n will offer to use the installed version — accept it. The logic still works. |
| An agent's output won't parse | The Code nodes already strip ```` ```json ```` fences and `try/catch`. If an agent gets chatty, lower the model temperature in the Gemini node options, or re-run. |
| Form/approval form won't open | The workflow must be running in **Test** mode (click Execute) so the trigger/wait forms are live. |
| Gemini model not found | In the Gemini node, set **Model** to `models/gemini-2.0-flash` or `models/gemini-1.5-flash`. |
