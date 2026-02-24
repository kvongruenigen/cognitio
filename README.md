# Cognitio

This repository contains a comprehensive cognitive test battery designed to assess various cognitive functions including memory, attention, executive function, and processing speed.

The general idea comes from the NIH Toolbox Cognition Battery and PsyToolkit.

## Overview

The app is an Electron + React desktop application with a local SQLite database. It supports multilingual UI (English/Spanish), configurable test settings, and session reporting.

## Getting started

- Install the beta version.
  - [Windows Installer](https://github.com/kvongruenigen/cognitio/releases/download/beta-testing/Cognitio.Setup.0.1.0-beta.exe)
  - [Mac Installer (ARM)](https://github.com/kvongruenigen/cognitio/releases/download/beta-testing/Cognitio-0.1.0-beta-arm64.dmg)
  - [Mac Installer (Intel)](https://github.com/kvongruenigen/cognitio/releases/download/beta-testing/Cognitio-0.1.0-beta.dmg)
- Close other heavy applications if possible.

## Settings & localization

- App language can be switched between English and Spanish.
- Test settings (PVT duration, Stroop keys, n-back parameters, Go/No-Go trial counts) are stored per session and shown in session results.

## Reports

- Participant PDF reports can be exported from the Results page.
- Reports include participant details, all sessions, summary statistics per test, and session-specific settings.

## Database

- SQLite is used for local persistence.
- Schema is defined in `src/electron/db/migrations.ts`.
- Session data and trials are stored in per-test tables and linked by session UUIDs.

## Tests included:

- **PVT (Psychomotor Vigilance Task)**: Assesses sustained attention.
- **Stroop Task**: Measures cognitive flexibility and processing speed.
- **N-Back Task**: Assesses working memory.
- **Corsi Span Task**: Evaluates visuospatial memory.
- **Digit Span Test**: Tests working memory.
- **Task Switching Test**: Measures cognitive flexibility.
- **Go/No-Go Task**: Evaluates response inhibition.
- **Stop Signal Task**: Variation of Go/No-Go Task.
- **Flanker Task**: Measures attention and inhibitory control.
