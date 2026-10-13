<p align="center">
  <img src="https://kvongruenigen.github.io/cognitio/assets/cognitio-mark.png" alt="Cognitio logo" width="120">
</p>

# Cognitio

Cognitio is a cognitive test battery for exploring attention, memory, executive function, and response speed. The desktop app combines ten tasks with participant management, configurable sessions, local data storage, and CSV and PDF exports.

**[Website](https://kvongruenigen.github.io/cognitio/)** · **[Try Cognitio Light](https://kvongruenigen.github.io/cognitio/play/)** · **[Latest release](https://github.com/kvongruenigen/cognitio/releases/latest)** · **[User guide](docs/USER_GUIDE.md)**

This repository distributes desktop releases and user documentation. The desktop application is built with Electron, React, TypeScript, and SQLite.

## Download the desktop app

Choose the installer for your system. These links point to the latest published release.

| System | Download |
| --- | --- |
| Windows, x64 | [Windows installer](https://github.com/kvongruenigen/cognitio/releases/latest/download/Cognitio.Setup.exe) |
| macOS, Apple Silicon | [Apple Silicon installer](https://github.com/kvongruenigen/cognitio/releases/latest/download/Cognitio-arm64.dmg) |
| macOS, Intel | [Intel installer](https://github.com/kvongruenigen/cognitio/releases/latest/download/Cognitio-x64.dmg) |
| Linux, x64 | [Linux AppImage](https://github.com/kvongruenigen/cognitio/releases/latest/download/Cognitio.AppImage) |

For previous versions and release notes, see [Releases](https://github.com/kvongruenigen/cognitio/releases).

## Try it in your browser

[Cognitio Light](https://kvongruenigen.github.io/cognitio/play/) runs all ten tasks without installation. Use a laptop or desktop with a keyboard.

- Run anonymous sessions without registering participants.
- Read task instructions and view results immediately.
- Download recorded trials as CSV.
- Switch between English, Spanish, and Russian.

Browser sessions stay in the page’s memory and disappear when you reload or close it. **Download your CSV before leaving.** Keep the tab visible during a task; switching away ends the session.

Use the desktop app for saved participant records, session history, and participant PDF reports.

## Getting started

1. Download and install the desktop app for your system.
2. Open **Settings** to choose a language and review the task parameters.
3. Select **Start Test**, then register or select a participant.
4. Choose a task and follow its instructions.
5. Open **Results** to review sessions and export data or a participant report.

Close other heavy applications before testing where possible. Each task provides its own response-key instructions.

## Included tasks

| Task | Focus |
| --- | --- |
| Psychomotor Vigilance Test (PVT) | Sustained attention and reaction speed over time |
| Stroop Test | Interference control and executive attention |
| N-Back Test | Working-memory updating |
| Digit Span Test | Verbal memory span |
| Corsi Span Test | Visuospatial memory span |
| Go/No-Go Test | Response inhibition |
| Stop Signal Task | Cancelling a prepared response after a stop signal |
| Flanker Test | Selective attention and conflict resolution |
| Task Switching Test | Cognitive flexibility between task rules |
| Task Switching Cued Test | Changing task rules in response to advance cues |

See [Test descriptions](docs/TEST_DESCRIPTIONS_BRIEF.md) for task structure and recorded data.

## Languages and settings

The app supports **English, Spanish, and Russian**. Change the interface language in **Settings**.

Available task settings include PVT duration, response keys, N-back parameters, and trial counts. The desktop app stores the settings used for each session and displays them with its results.

## Results and local storage

The desktop app stores participant records, sessions, and trials in a local SQLite database.

From **Results**, you can:

- Review session summaries and trial-level details.
- Export session and trial data as CSV.
- Export participant PDF reports with participant details, sessions, summary statistics, and session settings.

See the [User guide](docs/USER_GUIDE.md) for the full workflow.

## Documentation and feedback

- [User guide](docs/USER_GUIDE.md)
- [Test descriptions](docs/TEST_DESCRIPTIONS_BRIEF.md)
- [Testing guide](docs/TESTING_GUIDE.md)
- [Test cases](docs/TEST_CASES.md)
- [Changelog](docs/CHANGELOG.md)

To report a problem or suggest an improvement, [open an issue](https://github.com/kvongruenigen/cognitio/issues). Include your operating system, app version, task name, steps to reproduce, and expected and actual behavior.

## Background

The project draws inspiration from the NIH Toolbox Cognition Battery and PsyToolkit.
