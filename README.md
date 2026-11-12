# LangGraph-react: CopilotKit docs QA (LangGraph JS, React)

QA harness for the CopilotKit docs at https://docs.copilotkit.ai/langgraph-typescript. Every doc page gets a route in a real Next.js app with the doc's snippets pasted **verbatim**, and a screen-recorded clip shows what happens when a user follows the page. Where the docs break, the clip shows the break and the defect is listed in [FINDINGS.md](FINDINGS.md). Nothing here fixes a doc defect; the only allowed edits are marked `HARNESS` (rules: `.qa/instructions/QA-RULES.md`).

Status: recorded 2026-11-05, findings logged. 63 pages tracked in [assignment.json](assignment.json), 50 reachable (✅ 33 · ⚠️ 8 · ❌ 6 · N/A 1 · parent row 1 · Python-only 1), 48 clips, all on Loom. Current report: [report/status.html](report/status.html).

## Layout

```
agent/                   LangGraph JS graphs (langgraph.json), served by `langgraphjs dev` on 8050
frontend/                Next.js app, one route per doc page; Copilot Runtime as Next API routes (3050)
doc-snapshot/            rendered / .md snapshot of the docs (2026-11-05)
reference/showcase/      CopilotKit showcase source, used to confirm what a doc page leaves out
reference/verbatim/      doc versions of files the harness had to change (cited from HARNESS comments)
autorecorder/            clip recorder: cli.ts → core/engine.ts → actions/*.ts; config/*.config.ts; output videos/ (not committed)
tools/                   helper scripts (smoke, doc-check, clip-sheet)
assignment.json          per-page status, notes, findings, clip ids, Loom links
report/status.html       generated from assignment.json (paste-friendly, team Google Doc format)
FINDINGS.md              open doc defects (stable #N ids)
.qa/                     vendored shared tooling + rules (synced; don't edit here)
start-all.sh             starts the agent server and the frontend on the harness ports
```

Ports: frontend **3050**, agent **8050** (IPv6 `::1` only, so use `localhost`). Never 3000 / 8000. LangGraph-vue runs its own agent server (8152) and doesn't need this repo.

## Run

```sh
# keys: agent/.env (OPENAI_API_KEY) and frontend/.env.local (LANGGRAPH_DEPLOYMENT_URL), see .env.example
npm ci --prefix agent && npm ci --prefix frontend && npm ci --prefix autorecorder
./start-all.sh                       # logs in logs/
cd autorecorder
npm run doctor                       # must exit 0
npm run record:list
npm run record -- --pages=quickstart,frontend-tools
```
## How to review and report

- **A clip**: it should go doc → IDE (verbatim code) → demo, and for ❌/⚠️ also evidence (the faulty doc text drag-selected) → Notepad note. Nothing may be injected on the page.
- **A status**: ✅ works as documented, ⚠️ main feature works with a minor defect, ❌ main feature doesn't work as documented. If in doubt, ❌.
- **A finding**: add it to `FINDINGS.md` with its evidence (probe result, error text, frame). Format: QA-RULES section 5.
- **The report**: edit `assignment.json` (schema: `.qa/report/SCHEMA.md`), then rebuild. Don't hand-edit the HTML.

  ```sh
  python .qa/report/build.py assignment.json --out report/status.html
  # framework report for the team doc (needs a checkout of LangGraph-vue beside this repo)
  python .qa/report/build.py assignment.json ../LangGraph-vue/assignment.json --out LangGraph-testing-doc.html
  ```
