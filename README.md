# ChronoShield-WM

### Predictive Network Defence & Attack Progression Intelligence

**Smart India Hackathon 2026**

**Problem Statement ID:** SIH26153  
**Problem Statement:** AI based Network Attack Forecasting from Network Traffic Data  
**Organization:** National Technical Research Organisation (NTRO)  
**Category:** Software  
**Theme:** Blockchain & Cybersecurity  
**Team:** Rangers101

---

## Overview

**ChronoShield-WM** is an experimental network attack forecasting platform designed to answer a question beyond traditional detection:

> **Given everything observable right now, what malicious activity may happen next, which asset may be targeted, and within what time horizon?**

Traditional security systems primarily focus on detecting, correlating and investigating suspicious activity that is already observable.

ChronoShield-WM focuses on **future network-state and attack-progression forecasting**.

The proposed system represents an enterprise network as a changing directed graph, learns the temporal behaviour of individual hosts, rolls the learned state forward and produces auditable forecasts for future events, target assets and observable attack behaviours.

A central design principle is that a forecast must use **only information that was available before the forecast was issued**.

---

## The Problem

Network attacks often develop through multiple observable actions rather than appearing as one isolated event.

A defender may therefore want to know:

- Which host may be targeted next?
- What behaviour may occur next?
- How soon might it happen?
- What evidence caused the warning?
- How much warning time was obtained?
- How reliable is the prediction?

Forecasting this correctly is difficult because future information can easily leak into model inputs during training or evaluation.

ChronoShield-WM therefore treats **temporal integrity, auditability and future-data isolation** as core architectural requirements.

---

## Proposed Architecture

The proposed forecasting pipeline is:

```text
Network Telemetry
PCAP / Flows / Optional Identity Events
                ↓
Observation Ledger
                ↓
Availability-Time Filtering
                ↓
Past-Only Time Windows
                ↓
Directed Host Graph
                ↓
GraphSAGE Encoder
                ↓
Per-Host GRU Memory
                ↓
Latent Belief State
                ↓
Learned Transition Model
                ↓
K-Step Future Rollout
                ↓
Future-State / Event / Host / Behaviour Heads
                ↓
Calibration + Evidence Checks
                ↓
Immutable Forecast Record
                ↓
Future Reveal
                ↓
Independent Evaluation
```

---

## How the Model Works

### 1. Directed Network Graph

Hosts are represented as nodes and observed communication relationships are represented as directed edges.

Possible input features include:

- peer novelty;
- connection rate;
- internal fan-out;
- port diversity;
- protocol activity;
- directional traffic statistics;
- temporal activity patterns.

---

### 2. GraphSAGE

**GraphSAGE** captures relational context.

Instead of analysing one host independently, the model can learn from the communication behaviour of neighbouring systems.

---

### 3. GRU Temporal Memory

A shared **Gated Recurrent Unit (GRU)** maintains the historical state of each host.

This allows the model to distinguish a single unusual event from behaviour that has been developing over time.

---

### 4. Belief State

Graph and temporal information are combined into a compact learned representation of the currently observable network state.

This is an **observational belief state**.

It is not treated as perfect knowledge of attacker intent or as a complete digital twin of the enterprise.

---

### 5. Learned Transition Model

The transition model advances the current latent state into future latent states **without receiving actual future observations**.

Repeated transition steps create a **K-step future rollout**.

---

### 6. Forecast Heads

The rolled future states can support outputs such as:

- probability of a future qualifying malicious event;
- likely target host;
- observable behaviour class;
- predicted future traffic-state features;
- multiple forecast horizons.

Example horizons:

```text
+1 minute
+5 minutes
+15 minutes
```

---

## Strict Temporal Boundary

One of the most important parts of ChronoShield-WM is the separation between:

```text
PAST / OBSERVED
        │
        │ Forecast issued here
        ▼
FORECAST CUTOFF
        │
        │ No future information allowed
        ▼
HIDDEN FUTURE
        │
        ▼
OUTCOME REVEALED LATER
```

At the forecast cutoff, the system must not use:

- future packets;
- future graph edges;
- future victim identities;
- future attack labels;
- completed-flow values unavailable at the cutoff;
- future topology information;
- statistics fitted using test/future data.

Once issued, the original forecast remains unchanged.

---

## Immutable Forecast Record

Each prediction is stored as a forecast record containing information such as:

```text
Forecast ID
Forecast Origin
Maximum Input Time
Forecast Horizons
Target Host
Behaviour
Evidence References
Model / Schema Version
Forecast Mode
```

After the future outcome becomes available, it is scored against the original saved record.

The old forecast is never rewritten to match what happened later.

---

## Independent Future Scoring

Future outcomes are separated from inference.

```text
Frozen Forecast
       │
       ▼
Independent Replay Scorer
       ▲
       │
Withheld Future Outcome
```

The withheld future is used only after the forecast is frozen.

It must never become an input to the forecasting engine.

---

## MITRE ATT&CK Mapping

Supported behaviours may be mapped to **MITRE ATT&CK** techniques when sufficient evidence exists.

Example:

```text
Suspicious Remote-Service Interaction
                ↓
Possible ATT&CK Alignment
T1021.002 — SMB / Windows Admin Shares
                ↓
Lateral Movement
```

The mapping is intentionally evidence-bounded.

For example:

```text
Port 445 observed
```

does not automatically prove:

```text
Successful lateral movement
```

The system may return:

```text
Unknown / Insufficient Evidence
```

when the available telemetry does not justify a stronger interpretation.

---

## Evaluation Strategy

ChronoShield-WM is designed to compare increasingly complex models fairly.

```text
Logistic Regression
        ↓
Vector GRU
        ↓
GraphSAGE + GRU
        ↓
More complex temporal models only if justified
```

The simpler baseline receives the same:

- forecast origins;
- past-only information;
- target definitions;
- horizon definitions;
- evaluation intervals.

If a simpler model performs better, the architecture should be simplified rather than claiming unnecessary complexity.

---

## Planned Evaluation Metrics

Forecast evaluation may include:

### Classification

- Precision
- Recall
- F1 Score
- False Positive Rate
- PR-AUC

### Calibration

- Brier Score
- Expected Calibration Error
- Reliability analysis

### Early Warning

- Warning Lead Time
- Early-Warning Coverage
- False Alerts per Hour

### Target Forecasting

- Hit@K
- Recall@K

### Future-State Prediction

- MAE
- predictive skill relative to persistence

### System Performance

- inference latency;
- throughput;
- memory usage.

---

## Leakage Audit

Important evaluation rules include:

- no future packets;
- no future completed-flow values;
- no future network edges;
- no future victim lists;
- no attack labels as inference inputs;
- no test-set normalization leakage;
- no future teacher forcing during validation/test rollout;
- no rewriting a forecast after the outcome is known.

### Prefix-Invariance Test

A particularly important test is:

> If every record after the forecast cutoff is changed, the previously saved forecast at that cutoff must remain identical.

---

## Current Prototype

The current repository contains an **offline interactive simulation prototype** of the ChronoShield-WM workflow.

The prototype demonstrates:

- enterprise network topology;
- temporal network replay;
- evolving network signals;
- strict past/future separation;
- forecast cutoff locking;
- separate replay time;
- multi-horizon forecasting;
- likely target-host prediction;
- behaviour forecasting;
- evidence display;
- MITRE ATT&CK alignment;
- immutable forecast records;
- hidden-future reveal;
- forecast verification;
- simulated warning lead time;
- multiple scenarios;
- automatic demonstration mode.

---

## Important Prototype Note

The current browser prototype is a **deterministic simulation of the proposed forecasting workflow**.

Displayed forecast percentages are:

> **Illustrative simulation values**

They are **not presented as trained GraphSAGE-GRU accuracy or calibrated production probabilities**.

The current prototype demonstrates the temporal architecture, user workflow and forecast-before-reveal principle.

Training and measured evaluation of the forecasting models form the next implementation stage.

---

## Prototype Scenarios

### Developing Lateral Movement

Demonstrates unusual peer and remote-service behaviour developing over time.

The system freezes the information boundary, produces a future target/behaviour forecast and later reveals the simulated continuation.

---

### Benign Administrator Burst

Demonstrates that unusual fan-out does not automatically imply malicious progression.

Legitimate administrative behaviour may produce superficially suspicious network patterns.

---

### Possible Outbound Exfiltration

Demonstrates cautious interpretation of unusual outbound activity.

High outbound traffic alone is not treated as proof of data exfiltration.

---

## Technology Stack

### Current Prototype

- HTML
- CSS
- JavaScript
- SVG
- Offline browser execution

### Proposed Forecasting Implementation

- Python
- PyTorch
- PyTorch Geometric
- GraphSAGE
- GRU
- scikit-learn
- Pandas
- PyArrow
- FastAPI
- SQLite / PostgreSQL
- React
- Cytoscape.js
- Docker

---

## Key Design Contributions

ChronoShield-WM does not claim novelty simply because it combines graph and temporal neural networks.

The main system-level design contributions are:

- strict forecast information cutoffs;
- future-data isolation;
- immutable forecast records;
- measurable multi-horizon warnings;
- likely next-host forecasting;
- observable future-state decoding;
- separate attack-onset and continuation evaluation;
- observability-aware unknown/abstention behaviour;
- evidence-linked explanations;
- fair baseline comparison;
- measurable warning lead time.

---

## Running the Demo

Clone the repository:

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
```

Open the project folder.

Then open:

```text
index.html
```

in a modern desktop browser such as Google Chrome or Microsoft Edge.

The current prototype requires:

```text
No package installation
No backend
No external API
No internet connection
```

For a guided walkthrough, select:

```text
Auto Demo
```

---

## Current Status

| Component | Status |
|---|---|
| Interactive Prototype | ✅ Implemented |
| Offline Dashboard | ✅ Implemented |
| Replay Engine | ✅ Implemented |
| Forecast Cutoff / Future Isolation | ✅ Implemented |
| Forecast Record Workflow | ✅ Implemented |
| Evidence & ATT&CK Views | ✅ Implemented |
| Scenario Demonstrations | ✅ Implemented |
| Logistic Regression Baseline | 🔄 Planned / In Development |
| Vector GRU | 🔄 Planned / In Development |
| GraphSAGE-GRU Training | 🔄 Planned / In Development |
| Probability Calibration | 🔄 Planned |
| Cross-Dataset Evaluation | 🔄 Planned |

---

## Team

### Rangers101

Smart India Hackathon 2026

---

## Disclaimer

ChronoShield-WM is currently a research and prototype system developed for **Smart India Hackathon 2026**.

Unless explicitly identified as experimentally measured, displayed probabilities, warning times and scenario outcomes in the browser prototype are illustrative.

The system is intended exclusively for defensive cybersecurity research and authorized network environments.