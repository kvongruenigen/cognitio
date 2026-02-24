# TESTING GUIDE

## Purpose

This guide defines a reproducible manual test workflow for Cognitio.

Evidence references:

- Runtime/routes: `src/renderer/App.tsx`
- Session/trial persistence: `src/electron/ipc/handlers/sessions.ts`, `src/electron/ipc/handlers/trials.ts`
- Results/export behavior: `src/electron/ipc/handlers/results.ts`
- Telemetry behavior: `src/renderer/components/TelemetryTracker.tsx`, `src/electron/ipc/handlers/telemetry.ts`
- Data location: `src/electron/db/database.ts`

---

## Setup Checklist

## Environment

- OS target builds documented in repo:
  - macOS (`.dmg`)
  - Windows (`.msi` or portable)
  - Linux (`.AppImage`)
  - Source: `docs/app-instructions.md`
- Node.js 20+ for source runs.
  - Source: `docs/app-instructions.md`
- Install dependencies:
  - `npm install`
  - Source: `README.md`, `docs/app-instructions.md`
- Start app from source:
  - `npm run dev`
  - Source: `README.md`, `package.json`

## Build/packaging checks (optional for test leads)

- Build app: `npm run build`
- Package per OS: `npm run dist:mac`, `npm run dist:win`, `npm run dist:linux`
- Source: `README.md`, `package.json`

## Permissions and runtime assumptions

- App requests fullscreen during test runtime; it catches errors if fullscreen fails.
  - Source: `src/renderer/pages/TestRunner.tsx`
- No explicit OS-level permission prompts are implemented in app code for camera/microphone/location.
  - Source: searched code under `src/electron/**`, `src/renderer/**`

---

## Standard Test Session (End-to-End)

This is the baseline run every tester should complete at least once.

## Goal

Create participant -> run one test -> verify result detail -> create report -> export all results.

## Steps

1. Launch app (`npm run dev` or packaged build).
2. Go to `Settings` (`/settings`) and set:
   - `Training` disabled for faster run.
   - Source: constraints in `src/shared/config/settingsLimits.ts`
3. Go to `Participant Selection` (`/participant-selection`).
4. Create one participant in `Register Participant` (`/register-participant`).
5. Return to participant list, select participant, click `Start Assessment`.
6. In `Tests` (`/tests`), open `Go/No-Go` (`/tests/go-nogo`).
7. Complete trials to session end (do not abort).
8. Confirm return to `Tests` page.
9. Open `Results` (`/results`), select participant, open session details.
10. Verify session summary/trials load in `/results/:sessionUuid`.
11. Back to `/results`, click `Export PDF Report`, choose output folder
    - Verify the created PDF.
12. Click `Export All Results`, choose output folder.
13. Verify output files:
    - `results_sessions.csv`
    - `results_trials_long.csv`
    - `results-export-meta.json`
    - Source: `src/electron/ipc/handlers/results.ts`

## Expected end-to-end artifacts

- SQLite entries for participant/session/trials:
  - DB file: `<userData>/cognitive-tests.db`
  - Source: `src/electron/db/database.ts`
- Backup after session end:
  - `<userData>/backup/*.db`
  - `<userData>/backup/backup-manifest.jsonl`
  - Source: `src/electron/db/backup.ts`, `src/electron/db/backupManifest.ts`
- Export files in selected folder:
  - Source: `src/electron/ipc/handlers/results.ts`

---

## What to Observe During Testing

## Timing

- Phase start overlays should appear for `training` and `assessment` transitions.
  - Duration is `3000 ms`.
  - Source: `src/renderer/components/phase/constants.ts`, `src/renderer/components/phase/useAutoPhaseStartScreen.ts`
- Test runtime enters fullscreen and hides cursor.
  - Source: `src/renderer/pages/TestRunner.tsx`
- Summaries are computed from assessment trials (`trial_phase=assessment`) only.
  - Source: `src/shared/metrics/sessionSummary.ts`, `src/shared/metrics/trialPhase.ts`

## UI states

- Missing participant guard should redirect from test routes to participant selection.
  - Source: `src/renderer/pages/useRequiredParticipant.ts`
- `Tests` page should show “No participant selected” state if opened without route state.
  - Source: `src/renderer/pages/Tests.tsx`
- Results page should show loading, empty selection prompt, and action states (export/delete buttons).
  - Source: `src/renderer/pages/Results.tsx`, `src/renderer/components/results/ResultsDetailsPanel.tsx`

## Data saved

- Participant operations:
  - `participants:list/create/update/delete`
  - Source: `src/electron/ipc/handlers/participants.ts`
- Session operations:
  - `testSession:start/end/abort`
  - Source: `src/electron/ipc/handlers/sessions.ts`
- Trial writes per test:
  - Source: `src/electron/ipc/handlers/trials.ts`
- Clear-all destructive action:
  - Deletes all non-sqlite internal tables.
  - Source: `src/electron/ipc/handlers/data.ts`

## Export format

- “Export All Results” now writes compact analysis files (not per-table CSV dump):
  - `results_sessions.csv` (session-level + flattened summary metrics)
  - `results_trials_long.csv` (long-format trial rows + payload json)
  - `results-export-meta.json` (`formatVersion: 2`, counts, file list)
  - Source: `src/electron/ipc/handlers/results.ts`

---

## Feedback Rubric (Compact)

Use this format for every finding:

- Severity:
  - `S1 Critical`: data loss, crash, cannot continue core workflow
  - `S2 Major`: incorrect results, blocked sub-flow, export incorrect
  - `S3 Moderate`: UX/state inconsistency with workaround
  - `S4 Minor`: copy/layout polish
- Reproducibility:
  - `Always` / `Intermittent` / `Once`
- Expected vs actual:
  - One sentence expected behavior
  - One sentence actual behavior
- Context:
  - Route/screen
  - Participant code or session UUID (if relevant)
  - Timestamp
- Evidence:
  - Screenshot/video optional
  - Export file sample or relevant DB/telemetry record if available

Suggested issue template:

```md
Severity: S2
Reproducibility: Always
Screen/Route: /results
Expected: Export All writes compact files with metadata.
Actual: Only one file was written.
Context: Participant P001, session 1234..., 2026-02-23 14:20
Evidence: attached screenshot + output folder listing
```

---

## Quick Commands for Validation

- Electron transpile check: `npm run transpile:electron`
- Unit tests: `npm run test:unit`
- Build check: `npm run build`
- Source: `package.json`, `agents.md`

If any command fails, record:

- failing command
- error output snippet
- likely root cause
- minimal fix proposal

