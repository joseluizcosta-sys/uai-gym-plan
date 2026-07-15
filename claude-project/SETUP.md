# Setting up the UAI Dashboard as a claude.ai Project

## Steps (do these in the claude.ai web app or desktop app)
1. Go to **claude.ai → Projects → Create project**. Name it e.g. **"UAI 100K Dashboard"**.
2. Open the project → **Instructions / "Add instructions"** → paste the full instruction
   block from `PROJECT_INSTRUCTIONS.md` (everything under the `---`).
3. **Add to project knowledge** (upload these files):
   - `CLAUDE.md` (repo root) — the editing conventions.
   - `garmin_metrics.json` (repo root) — current Garmin/recovery data.
   - `weight_data.json` (repo root) — current weight history.
   - *(optional)* `Main Vault/Health/exams/ergoespirometria-2026-05-29.md` — full CPET.
4. *(Optional)* Connect the **GitHub connector** in the project so it can read the live
   repo files instead of relying on the uploaded snapshots. Note: even connected, the
   claude.ai chat writes proposed changes back only if you approve via the connector's
   own flow; nightly automated updates still run through Claude Code locally.
5. Start a chat inside the project — it now has all context loaded.

## Keeping the knowledge fresh
The uploaded JSONs are point-in-time snapshots. Two options:
- **Manual:** re-upload `garmin_metrics.json` / `weight_data.json` when you want the
  Project to reason about the latest data, or paste them into the chat.
- **GitHub connector (step 4):** the Project reads the live files each session — no
  re-uploading. Recommended, since the nightly job keeps the repo current.

## What stays in Claude Code (not this Project)
- Fetching Garmin (the Garmin MCP is local only).
- Editing the 14 MB `index.html` and `git push` — via the `/update-dashboard` skill
  and the `uai_daily_update` launchd job.
Use this Project to think/plan/draft; use Claude Code to fetch/apply/publish.
