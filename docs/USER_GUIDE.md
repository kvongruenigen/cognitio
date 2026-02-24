## Scope

This guide documents current user flows in the app based on repository code.

- Verified against routes/components/handlers in:
  - `~/src/renderer/App.tsx`
  - `~/src/renderer/pages/*`
  - `~/src/renderer/components/*`
  - `~/src/electron/ipc/handlers/*`
  - `~/src/electron/db/migrations.ts`

---

## Flow 1: Open App and Navigate

### Screen: Home (`/`)

#### Purpose:
  - Entry screen to start participant workflow or view results.

#### How to use:
  - Select **Start Test** to navigate to participant selection.
  - Select **Results** to navigate to results.
  - If development mode is enabled, a telemetry card is shown.

#### Inputs:
  - Button clicks on navigation cards.

#### Outputs:
  - Route navigation to `/participant-selection`, `/results`, and conditional `/telemetry`.

#### Error states:
  - No user-facing error handling in this screen code.
  - **TODO:** Verify whether runtime navigation errors are surfaced anywhere in UI.

#### Data saved:
  - No direct data writes on this screen.

#### Evidence:
  - `~/src/renderer/pages/Home.tsx`
  - `~/src/renderer/App.tsx`

---

## Flow 2: Create or Manage Participants

### Screen: Participant Selection (`/participant-selection`)

#### Purpose:
  - Browse participants, search/select one, start assessments, edit participant, delete participant.

#### How to use:
  - Use search input to filter by code/first name/last name.
  - Click participant row to select.
  - Use toolbar:
    - **Register New Participant*~**
  - Delete selected participant from details actions.

#### Inputs:
  - Search text, participant row selection, action buttons, delete/edit confirmations.

#### Outputs:
  - List and stats update.
  - Navigation:
    - `/register-participant`
    - `/tests` with selected participant in route state.
  - Participant updates/deletes reflected immediately in UI state.

#### Error states:
  - Missing preload API logs console error.
  - Delete failure shows alert (`participantSelection.delete.failed`).
  - Edit validation errors for required fields, invalid/future/unrealistic DOB.
  - Update collision shows code-exists error.

#### Data saved:
  - Read participants: `participants:list`.
  - Update participant: `participants:update`.
  - Delete participant (and cascaded sessions): `participants:delete`.

#### Evidence:
  - `~/src/renderer/pages/ParticipantSelection.tsx`
  - `~/src/renderer/components/participantSelection/ParticipantSelectionActionToolbar.tsx`
  - `~/src/renderer/components/participantSelection/ParticipantSelectionListPanel.tsx`
  - `~/src/electron/ipc/handlers/participants.ts`
  - `~/src/electron/db/migrations.ts`

### Screen: Register Participant (`/register-participant`)

#### Purpose:
  - Create a new participant record.

#### How to use:
  - Fill required fields: participant code, first name, last name, DOB, gender.
  - Submit form to create participant, then return to participant selection.

#### Inputs:
  - Form fields and submit action.

#### Outputs:
  - On success, route navigation to `/participant-selection`.
  - Inline field/form errors on invalid input or backend failure.

#### Error states:
  - Required-field errors.
  - DOB format/validity/future/unrealistic checks.
  - Duplicate participant code (`PARTICIPANT_CODE_EXISTS`).
  - Generic create failure (`register.errors.failed`).

#### Data saved:
  - Creates participant via `participants:create`.
  - Participant code is normalized with configured prefix before save.

#### Evidence:
  - `~/src/renderer/pages/RegisterParticipant.tsx`
  - `~/src/electron/ipc/handlers/participants.ts`
  - `~/src/electron/config/deviceConfig.ts`

---

## Flow 3: Select and Start a Test Session

### Screen: Tests (`/tests`)

#### Purpose:
  - Show available cognitive tests and current settings summary before launch.

#### How to use:
  - Enter this screen with a selected participant.
  - Pick one test card to start that test route.

#### Inputs:
  - Participant passed through route state.
  - Test card click.

#### Outputs:
  - Navigation to a test route, for example:
    - `/tests/pvt`
    - `/tests/stroop`
    - `/tests/digit-span`
    - `/tests/go-nogo`
    - `/tests/n-back`
    - `/tests/stop-signal`
    - `/tests/task-switching`
    - `/tests/task-switching-cued`
    - `/tests/flanker`
    - `/tests/corsi-span`

#### Error states:
  - If no participant is in route state, user sees “No participant selected” and can return to participant selection.

#### Data saved:
  - No DB writes on this selection screen.

#### Evidence:
  - `~/src/renderer/pages/Tests.tsx`
  - `~/src/renderer/App.tsx`
  - `~/src/renderer/pages/testPages/index.tsx`

### Screen Group: Test Runtime Pages (`/tests/*`)

#### Purpose:
  - Run each cognitive task, record trials, and end/abort sessions.

#### How to use:
  - Enter a test route from `/tests`.
  - Complete task-specific interactions; on completion return to `/tests`.
  - Aborting returns to `/tests`.

#### Inputs:
  - Task interactions (key presses, clicks, etc.) defined by each test component.

#### Outputs:
  - Session lifecycle operations:
    - start session
    - save trial rows
    - end session on completion
    - abort session on early exit
  - Navigation back to `/tests`.

#### Error states:
  - Participant context is required; missing participant redirects to `/participant-selection`.
  - **TODO: Verify** complete per-test user-facing error messages; most handlers use async calls without explicit UI error rendering in all test components.

#### Data saved:
  - Session records in `test_sessions`.
  - Trial records in task-specific tables:
    - `pvt_trials`
    - `stroop_trials`
    - `digit_span_trials`
    - `go_nogo_trials`
    - `n_back_trials`
    - `stop_signal_trials`
    - `flanker_trials`
    - `task_switching_trials`
    - `corsi_trials`
  - Session status transitions (`in_progress`, `completed`, `aborted`).

#### Evidence:
  - `~/src/renderer/pages/testPages/createStandardTestPage.tsx`
  - `~/src/renderer/pages/useRequiredParticipant.ts`
  - `~/src/renderer/components/PVT.tsx`
  - `~/src/renderer/components/StroopTest.tsx`
  - `~/src/renderer/components/DigitSpan.tsx`
  - `~/src/renderer/components/GoNoGo.tsx`
  - `~/src/renderer/components/nBack.tsx`
  - `~/src/renderer/components/StopSignal.tsx`
  - `~/src/renderer/components/FlankerTest.tsx`
  - `~/src/renderer/components/TaskSwitchingTest.tsx`
  - `~/src/renderer/components/CorsiSpan.tsx`
  - `~/src/electron/ipc/handlers/sessions.ts`
  - `~/src/electron/ipc/handlers/trials.ts`
  - `~/src/electron/db/migrations.ts`

---

## Flow 4: View Session Results and Export Data

### Screen: Results (`/results`)

#### Purpose:
  - Select participant, review session list/status, open session details, delete session(s), export data.

#### How to use:
  - Select a participant from left panel.
  - Inspect session list.
  - Use actions:
    - **Export All Results**
    - **Export Participant PDF**
    - **Delete Session**
    - **Delete All Sessions**
    - **View Details**

#### Inputs:
  - Participant selection, search text, action buttons, confirm dialogs.

#### Outputs:
  - Session list updates for deletes.
  - Navigation to `/results/:sessionUuid`.
  - Export workflows via native file dialogs.

#### Error states:
  - Missing preload API logs console error.
  - Participant-PDF export error displays inline error text.
  - Delete failures show alert (`results.delete.failed`).
  - Export-all failure logs console error.

#### Data saved:
  - Reads participants and participant results.
  - Deletes session(s) when requested.
  - Creates exported files:
    - `results_sessions.csv`
    - `results_trials_long.csv`
    - `results-export-meta.json`
  - Participant report export to PDF via save dialog.

#### Evidence:
  - `~/src/renderer/pages/Results.tsx`
  - `~/src/renderer/components/results/ResultsActionToolbar.tsx`
  - `~/src/renderer/components/results/ResultsDetailsPanel.tsx`
  - `~/src/electron/ipc/handlers/results.ts`

### Screen: Session Result Details (`/results/:sessionUuid`)

#### Purpose:
  - Show one session’s metadata, summary metrics, trial-level details, and parsed settings.

#### How to use:
  - Open from **View Details** in Results table.
  - Use back action to return to `/results`.
  - Optional delete of current session.

#### Inputs:
  - Route parameter `sessionUuid`.
  - Optional route state (`selectedParticipantId`, `participantCode`).
  - Delete confirmation.

#### Outputs:
  - Displays computed summary and settings rows.
  - Deletes session and returns to results on success.

#### Error states:
  - Missing session UUID.
  - API unavailable (`getSessionDetails` missing).
  - Session not found.
  - Generic session load failure.
  - Delete failure alert (`results.delete.failed`).

#### Data saved:
  - No new analytics data saved here.
  - Deleting session removes persisted session/trials.

#### Evidence:
  - `~/src/renderer/pages/SessionResult.tsx`
  - `~/src/renderer/components/sessionResult/SessionResultView.tsx`
  - `~/src/electron/ipc/handlers/results.ts`
  - `~/src/shared/metrics/sessionSummary.ts`
  - `~/src/shared/metrics/sessionSettings.ts`

---

## Flow 5: Configure App Behavior

### Screen: Settings (`/settings`)

#### Purpose:
  - Configure language, training/development flags, task settings, key bindings, and destructive maintenance actions.

#### How to use:
  - Adjust settings controls in grouped sections.
  - Reset settings to defaults.
  - Clear all persisted app data (confirmation required).

#### Inputs:
  - Form controls (selects/checkboxes/number inputs/buttons), keyboard/mouse capture for PVT input binding.

#### Outputs:
  - Immediate in-app setting updates.
  - Reset to defaults.
  - Optional full data clear, then navigation to `/`.

#### Error states:
  - Data-clear failure shows alert (`settings.danger.clearFailed`).
  - Success alert on clear (`settings.danger.clearSuccess`).

#### Data saved:
  - Settings persisted to browser `localStorage` (`app_settings`).
  - Device-config values are loaded and merged into active settings.
  - Clear-all operation deletes all DB tables and resets sqlite sequence.

#### Evidence:
  - `~/src/renderer/pages/Settings.tsx`
  - `~/src/renderer/settings.tsx`
  - `~/src/electron/ipc/handlers/data.ts`
  - `~/src/electron/config/deviceConfig.ts`

---

## Flow 6: Review Runtime Telemetry (Development Mode)

### Screen: Telemetry (`/telemetry`) (conditional)

#### Purpose:
  - Inspect runtime events/crashes/transitions and export telemetry CSV.

#### How to use:
  - Enable development mode in Settings.
  - Open `/telemetry`.
  - Filter by session/event type/level/search.
  - Export filtered telemetry rows.

#### Inputs:
  - Filter controls, paging controls, export button.

#### Outputs:
  - Filtered/paginated telemetry list.
  - CSV export through native save dialog.

#### Error states:
  - API unavailable messages when telemetry preload APIs are missing.
  - Export unavailable message on export API failure.

#### Data saved:
  - Viewing page does not save user data by itself.
  - Telemetry events are continuously produced by tracker + main process.
  - Export writes CSV file chosen by user.

#### Evidence:
  - `~/src/renderer/pages/Telemetry.tsx`
  - `~/src/renderer/components/TelemetryTracker.tsx`
  - `~/src/electron/ipc/handlers/telemetry.ts`
  - `~/src/electron/db/telemetry.ts`

---

## Shared Data Persistence Reference

#### Participants:
  - Table: `participants`
  - Evidence: `~/src/electron/db/migrations.ts`

#### Sessions:
  - Table: `test_sessions`
  - Evidence: `~/src/electron/db/migrations.ts`

#### Trials:
  - Tables listed in Flow 3 runtime section
  - Evidence: `~/src/electron/db/migrations.ts`

#### Telemetry:
  - Table: `app_telemetry_events`
  - Evidence: `~/src/electron/db/migrations.ts`

#### Local settings:
  - `localStorage["app_settings"]`
  - Evidence: `~/src/renderer/settings.tsx`

#### Device config source:
  - Repo `device-config.json` preferred, fallback under Electron `userData`
  - Evidence: `~/src/electron/config/deviceConfig.ts`

---

## Open Verification Items

- **TODO: Verify** whether every test component surfaces start/save/end failures to users consistently, or only logs errors in some components.
- **TODO: Verify** exact user-facing labels for all per-test runtime buttons/messages in every locale (`en`, `es`) beyond shared labels already referenced.
