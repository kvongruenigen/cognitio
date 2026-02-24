## Conventions

- IDs: `TC-001`..`TC-020`
- Core persistence location for most cases:
  - DB: `<userData>/cognitive-tests.db` (`src/electron/db/database.ts`)
  - Backup: `<userData>/backup` + `backup-manifest.jsonl` (`src/electron/db/database.ts`, `src/electron/db/backup.ts`)
- Telemetry storage:
  - `app_telemetry_events` table (`src/electron/db/migrations.ts`, `src/electron/db/telemetry.ts`)

---

## TC-001 App Launch and Route Shell

- Preconditions:
  - Dependencies installed (`npm install`).
- Steps:
  1. Run `npm run dev`.
  2. Wait for app to open to `/`.
- Expected Result:
  - Home screen renders and navigation is available.
- Notes:
  - Verify no blocking startup error dialogs.
- Logs/Results Storage:
  - Telemetry `app_started` + renderer start events (`src/electron/main.ts`, `src/renderer/components/TelemetryTracker.tsx`).

## TC-002 Register Participant Required Validation

- Preconditions:
  - App running at `/register-participant`.
- Steps:
  1. Leave required fields empty.
  2. Click submit.
- Expected Result:
  - Required-field errors shown; participant not created.
- Notes:
  - Validate code/first/last/dob/gender all enforced.
- Logs/Results Storage:
  - No new participant row in `participants` table (`src/electron/db/migrations.ts`, `src/electron/ipc/handlers/participants.ts`).

## TC-003 Register Participant DOB Validation

- Preconditions:
  - On `/register-participant`.
- Steps:
  1. Enter invalid DOB format or future date.
  2. Submit.
- Expected Result:
  - DOB-specific validation error shown; no record saved.
- Notes:
  - Check both invalid format and future date paths.
- Logs/Results Storage:
  - No participant insert (`src/renderer/pages/RegisterParticipant.tsx`, `src/electron/ipc/handlers/participants.ts`).

## TC-004 Register Duplicate Participant Code

- Preconditions:
  - Existing participant with code `P001`.
- Steps:
  1. Create second participant using same code.
- Expected Result:
  - Duplicate code error shown (`codeExists` path).
- Notes:
  - Backend maps sqlite unique constraint to app error.
- Logs/Results Storage:
  - Unique constraint handling in `participants:create` (`src/electron/ipc/handlers/participants.ts`).

## TC-005 Edit Participant Success

- Preconditions:
  - At least one participant exists.
- Steps:
  1. Select participant in `/participant-selection`.
  2. Click `Edit`, update one field, confirm/save.
- Expected Result:
  - Updated values shown in participant list.
- Notes:
  - Confirm modal flow and validation.
- Logs/Results Storage:
  - Updated row in `participants` table (`src/renderer/pages/ParticipantSelection.tsx`, `src/electron/ipc/handlers/participants.ts`).

## TC-006 Delete Participant Cascade

- Preconditions:
  - Participant exists with one or more sessions.
- Steps:
  1. Select participant.
  2. Delete participant and confirm.
- Expected Result:
  - Participant removed from list.
  - Related sessions/trials are removed via FK cascade.
- Notes:
  - Validate by searching participant in Results after deletion.
- Logs/Results Storage:
  - FK cascade schema (`src/electron/db/migrations.ts`), delete handler (`src/electron/ipc/handlers/participants.ts`).

## TC-007 Tests Page Guard Without Participant

- Preconditions:
  - Navigate directly to `/tests` without route state.
- Steps:
  1. Open `/tests` from URL/hash.
- Expected Result:
  - “No participant selected” state displayed with back action.
- Notes:
  - This verifies route-state dependency.
- Logs/Results Storage:
  - Renderer-only UI state (`src/renderer/pages/Tests.tsx`).

## TC-008 Start and Complete Go/No-Go Session

- Preconditions:
  - Participant selected; in `/tests`.
- Steps:
  1. Open `/tests/go-nogo`.
  2. Complete run normally.
- Expected Result:
  - Session starts and ends.
  - Returns to `/tests`.
  - Session appears in `/results` for participant.
- Notes:
  - Use minimal trial counts from settings to keep run short.
- Logs/Results Storage:
  - `test_sessions` + `go_nogo_trials` tables (`src/electron/ipc/handlers/sessions.ts`, `src/electron/ipc/handlers/trials.ts`, `src/electron/db/migrations.ts`).

## TC-009 Abort Session Path

- Preconditions:
  - Participant selected, test started.
- Steps:
  1. Trigger test abort path (per test’s back/exit behavior).
- Expected Result:
  - `testSession:abort` called; status becomes `aborted`.
  - Returns to `/tests`.
- Notes:
  - Validate in `/results` status badge.
- Logs/Results Storage:
  - Session status update in `test_sessions` (`src/electron/ipc/handlers/sessions.ts`).

## TC-010 Phase Start Overlay Timing

- Preconditions:
  - Training enabled and test with phase transitions.
- Steps:
  1. Start a test that supports training + assessment.
  2. Observe phase transition overlays.
- Expected Result:
  - Overlay displays `training`/`assessment` screen for 3000 ms.
- Notes:
  - Verify overlay appears once per phase transition.
- Logs/Results Storage:
  - Phase UI logic (`src/renderer/components/phase/useAutoPhaseStartScreen.ts`, `src/renderer/components/phase/constants.ts`).

## TC-011 Results Participant Search and Selection

- Preconditions:
  - Multiple participants exist.
- Steps:
  1. Open `/results`.
  2. Search by code/first/last name.
  3. Select participant.
- Expected Result:
  - Filtered list updates.
  - Selected participant sessions load.
- Notes:
  - Validate loading state appears while fetching results.
- Logs/Results Storage:
  - Results read path (`src/renderer/pages/Results.tsx`, `src/electron/ipc/handlers/results.ts`).

## TC-012 Open Session Details

- Preconditions:
  - Participant has at least one session.
- Steps:
  1. In `/results`, click `View Details` on a session.
- Expected Result:
  - Navigates to `/results/:sessionUuid`.
  - Summary/settings/trials render.
- Notes:
  - Back action should restore selected participant when state exists.
- Logs/Results Storage:
  - Session details API + summary build (`src/renderer/pages/SessionResult.tsx`, `src/electron/ipc/handlers/results.ts`, `src/shared/metrics/sessionSummary.ts`).

## TC-013 Delete Single Session From Results

- Preconditions:
  - Participant with one or more sessions.
- Steps:
  1. Delete one session from `/results` (or session detail page).
- Expected Result:
  - Session removed from list/detail after confirm.
- Notes:
  - Failure should show `results.delete.failed`.
- Logs/Results Storage:
  - Delete handlers (`src/renderer/pages/Results.tsx`, `src/renderer/pages/SessionResult.tsx`, `src/electron/ipc/handlers/results.ts`).

## TC-014 Delete All Sessions for Selected Participant

- Preconditions:
  - Participant has multiple sessions.
- Steps:
  1. In `/results`, click delete-all sessions and confirm.
- Expected Result:
  - Session list clears for selected participant.
- Notes:
  - Verify participant remains, only sessions removed.
- Logs/Results Storage:
  - `results:deleteParticipantSessions` path (`src/electron/ipc/handlers/results.ts`).

## TC-015 Export All Results Format

- Preconditions:
  - At least one session exists.
- Steps:
  1. Click `Export All Results`.
  2. Choose folder.
- Expected Result:
  - Output folder contains:
    - `results_sessions.csv`
    - `results_trials_long.csv`
    - `results-export-meta.json`
  - Metadata contains `formatVersion: 2`.
- Notes:
  - CSVs begin with `# app_version` and `# exported_at` header lines.
- Logs/Results Storage:
  - Export implementation (`src/electron/ipc/handlers/results.ts`).

## TC-016 Export Participant PDF

- Preconditions:
  - Selected participant with at least one session.
- Steps:
  1. Click `Export PDF`.
  2. Choose save path.
- Expected Result:
  - PDF saved to selected location.
  - UI shows exporting state and returns to idle.
- Notes:
  - If no sessions, button disabled.
- Logs/Results Storage:
  - PDF export path (`src/renderer/components/results/ResultsDetailsPanel.tsx`, `src/electron/ipc/handlers/results.ts`).

## TC-017 Clear All Data

- Preconditions:
  - Existing participants/sessions present.
- Steps:
  1. Open `/settings`.
  2. Trigger `Clear All Data` and confirm.
- Expected Result:
  - Success alert shown.
  - Settings reset.
  - Navigates to `/`.
  - Participants/results cleared.
- Notes:
  - Confirm destructive action requires confirmation prompt.
- Logs/Results Storage:
  - Clear-all transaction (`src/renderer/pages/Settings.tsx`, `src/electron/ipc/handlers/data.ts`).

## TC-018 Telemetry Route Visibility Gate

- Preconditions:
  - Development mode initially disabled.
- Steps:
  1. Verify `/telemetry` route is not shown in normal navigation.
  2. Enable development mode in Settings.
  3. Re-open navigation/home and access telemetry.
- Expected Result:
  - Telemetry route available only when `developmentModeEnabled` is true.
- Notes:
  - Route guard implemented in app router, not server-side auth.
- Logs/Results Storage:
  - Conditional route logic (`src/renderer/App.tsx`, `src/renderer/settings.tsx`).

## TC-019 Telemetry Filtering and CSV Export

- Preconditions:
  - Development mode enabled and telemetry events present.
- Steps:
  1. Open `/telemetry`.
  2. Apply filters (session/event/level/search).
  3. Export CSV.
- Expected Result:
  - Event list reflects filters.
  - Export writes CSV file with metadata header lines.
- Notes:
  - Empty export path returns `{ canceled: false, filePath: null }` when no rows.
- Logs/Results Storage:
  - Telemetry APIs (`src/renderer/pages/Telemetry.tsx`, `src/electron/ipc/handlers/telemetry.ts`).

## TC-020 Backup Written on Session End

- Preconditions:
  - Complete at least one session.
- Steps:
  1. Finish a test session normally.
  2. Inspect backup directory.
- Expected Result:
  - New backup `.db` file exists.
  - `backup-manifest.jsonl` appended with session entry.
- Notes:
  - Backup occurs after `testSession:end`.
- Logs/Results Storage:
  - Backup implementation and manifest format (`src/electron/ipc/handlers/sessions.ts`, `src/electron/db/backup.ts`, `src/electron/db/backupManifest.ts`).

---

## Open Verification Items

- **TODO: Verify** exact packaged installer extension used on Windows release pipeline (`.msi` vs other targets) for current release process; docs mention `.msi` while `electron-builder.json` also includes `portable` + `nsis`.
- **TODO: Verify** if local OS security policies can block fullscreen request on all target environments.
