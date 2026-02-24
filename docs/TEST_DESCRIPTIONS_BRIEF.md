# General Session Architecture

All trial tables reference:

`test_sessions(uuid)`

Each trial table additionally includes:

- `session_uuid TEXT NOT NULL`
- `trial_index INTEGER NOT NULL`
- `created_at DATETIME DEFAULT CURRENT_TIMESTAMP`
- `trial_phase TEXT NOT NULL DEFAULT 'assessment'`

The `trial_phase` column allows explicit separation of training and assessment trials.

---

# Psychomotor Vigilance Test (PVT)

## Introduction

The Psychomotor Vigilance Test (PVT) measures sustained attention and behavioral alertness. 
It is particularly sensitive to sleep deprivation and fatigue-related cognitive slowing. 
Participants respond to visual stimuli appearing at unpredictable intervals, allowing precise quantification of reaction time variability, lapses, and impulsive false starts.

## Task Structure

- Variable inter-stimulus delay
- Single stimulus detection
- Immediate response required
- No corrective feedback during assessment phase

## Trial-Level Data Output

Each trial records stimulus timing, reaction metrics, cumulative performance indicators, and response classification.

## Database Schema

Table: `pvt_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- status TEXT NOT NULL  ("correct" | "too_early" | "no_response")  
- random_delay_ms INTEGER NOT NULL  
- time_since_start_ms INTEGER NOT NULL  
- rt_since_start_ms INTEGER  
- rt_since_stimulus_ms INTEGER  
- mean_rt_so_far_ms REAL  
- lapses_so_far INTEGER NOT NULL  
- false_starts_so_far INTEGER NOT NULL  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# Stroop Color–Word Task

## Introduction

The Stroop paradigm assesses cognitive interference and executive control. 
Performance differences between congruent and incongruent trials reflect the cost of suppressing automatic reading processes in favor of color naming.

## Task Structure

- Color word presented in colored ink
- Congruent or incongruent condition
- Forced-choice color response
- Reaction time and accuracy recorded

## Trial-Level Data Output

- Stimulus word
- Ink color
- Congruency classification
- Response mapping
- Accuracy
- Reaction time

## Database Schema

Table: `stroop_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- word TEXT NOT NULL  
- color TEXT NOT NULL  
- congruent INTEGER NOT NULL  
- response TEXT  
- correct INTEGER NOT NULL  
- rt_ms INTEGER  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# Digit Span Task

## Introduction

The Digit Span task measures verbal short-term memory capacity. 
Span length reflects the maximum number of sequential digits that can be accurately maintained and reproduced.

## Task Structure

- Sequential digit presentation
- Serial recall input
- Span increases following correct responses
- Termination after repeated errors

## Trial-Level Data Output

- Span length
- Presented sequence (JSON)
- Entered sequence (JSON)
- Reaction times per digit (JSON)
- Accuracy flag

## Database Schema

Table: `digit_span_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- span_length INTEGER NOT NULL  
- sequence_presented TEXT NOT NULL  
- sequence_entered TEXT NOT NULL  
- rts TEXT NOT NULL  
- correct INTEGER NOT NULL  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# Go/No-Go Task

## Introduction

The Go/No-Go paradigm evaluates inhibitory control under response bias. 
Frequent go trials create a prepotent response tendency that must be suppressed during infrequent no-go trials.

## Task Structure

- Trial type: "go" or "no_go"
- Fixed response window
- Accuracy classification
- Inter-trial interval logged

## Trial-Level Data Output

- Trial type
- Reaction time
- Status classification
- Inter-trial interval
- Previous trial status
- Total presses within trial

## Database Schema

Table: `go_nogo_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- trial_type TEXT NOT NULL  
- rt INTEGER  
- status TEXT NOT NULL  
- iti INTEGER  
- previous_status TEXT  
- total_presses INTEGER NOT NULL  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# N-Back Task (2-Back)

## Introduction

The 2-Back task measures working memory updating and continuous monitoring. 
Participants respond when the current stimulus matches the one presented two trials earlier.

## Task Structure

- Blocked design
- Match vs non-match trials
- Continuous stimulus stream
- Error-type classification

## Trial-Level Data Output

- Block number
- Trial index
- Trial type (match / non-match)
- Overall score
- Match flag
- Miss flag
- False alarm flag
- Reaction time
- Current and preceding stimuli identifiers

## Database Schema

Table: `n_back_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- block INTEGER NOT NULL  
- trial_index INTEGER NOT NULL  
- trial_type INTEGER NOT NULL  
- score INTEGER NOT NULL  
- match INTEGER DEFAULT 0  
- miss INTEGER DEFAULT 0  
- false_alarm INTEGER DEFAULT 0  
- rt INTEGER  
- memory INTEGER  
- current_letter INTEGER  
- nback1 INTEGER  
- nback2 INTEGER  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# Stop Signal Task

## Introduction

The Stop Signal Task quantifies response inhibition after movement initiation. 
Participants execute rapid responses unless a stop signal appears after a variable delay. 
The paradigm enables estimation of inhibitory control dynamics.

## Task Structure

- Go and Stop trials
- Variable stop-signal delay
- Explicit response classification
- Training and main block separation

## Trial-Level Data Output

- Block identifier
- Trial type
- Required response
- Stop-signal delay
- Response key
- Reaction time
- Response status
- Early response indicator

## Database Schema

Table: `stop_signal_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- block INTEGER NOT NULL  
- trial_type TEXT NOT NULL  
- required_response TEXT NOT NULL  
- stop_signal_delay_ms INTEGER NOT NULL  
- response_key TEXT  
- response_rt_ms INTEGER  
- response_status TEXT NOT NULL  
- responded_before_stop INTEGER NOT NULL  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# Flanker Task

## Introduction

The Flanker paradigm assesses selective attention and interference resolution. 
Reaction time differences between congruent and incongruent flankers quantify cognitive control efficiency.

## Task Structure

- Central target arrow
- Surrounding flankers
- Congruency classification
- Directional response

## Database Schema

Table: `flanker_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- target_direction TEXT NOT NULL  
- flanker_direction TEXT NOT NULL  
- congruent INTEGER NOT NULL  
- response TEXT  
- correct INTEGER NOT NULL  
- rt_ms INTEGER  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# Task Switching Paradigm

## Introduction

Task switching paradigms measure cognitive flexibility and rule reconfiguration. 
Switch trials typically incur performance costs relative to repeat trials.

## Task Structure

- Paradigm type: alternate_runs | cued
- Multiple task rules
- Switch vs repeat trials
- Congruency where applicable

## Database Schema

Table: `task_switching_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- paradigm TEXT NOT NULL  
- task_type TEXT NOT NULL  
- switch_trial INTEGER NOT NULL  
- congruent INTEGER  
- stimulus_primary TEXT NOT NULL  
- stimulus_secondary TEXT NOT NULL  
- required_response TEXT NOT NULL  
- response_key TEXT  
- status TEXT NOT NULL  
- correct INTEGER NOT NULL  
- rt_ms INTEGER  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

# Corsi Block-Tapping Task

## Introduction

The Corsi task measures visuospatial working memory capacity. 
Participants reproduce spatial sequences of highlighted blocks. 
Performance reflects spatial span and error dynamics.

## Task Structure

- Spatial grid presentation
- Sequential highlighting
- Serial reproduction
- Termination after repeated errors

## Database Schema

Table: `corsi_trials`

- id INTEGER PRIMARY KEY AUTOINCREMENT  
- session_uuid TEXT NOT NULL  
- trial_index INTEGER NOT NULL  
- span_length INTEGER NOT NULL  
- sequence_shown TEXT NOT NULL  
- sequence_clicked TEXT NOT NULL  
- clicked_count INTEGER NOT NULL  
- correct INTEGER NOT NULL  
- errors_in_row INTEGER NOT NULL  
- grid_positions TEXT NOT NULL  
- trial_phase TEXT NOT NULL DEFAULT 'assessment'  
- created_at DATETIME DEFAULT CURRENT_TIMESTAMP  

---

End of Document.
