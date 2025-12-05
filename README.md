# Mammon

Mammon is a multi-engine trading framework built around modular Pine Script components.  
Each engine is a pure signal generator (enter/exit only). Higher-level layers handle
aggregation, risk, and pyramiding.

## Architecture

At a high level:

- **Engines**  
  Five families of engines, each with 4 sub-engines (two angles × long/short).  
  Every sub-engine:
  - Outputs a **0–100 signal score** (or discrete enter/exit signal).
  - Has its own **kill switch** based on local P&L and conditions.
  - Reports its **on/off / killed** state to the Judge.

- **Judge**  
  Aggregates engine scores and states, producing a single **per-side score**
  (long vs short) for the orchestrator.

- **Orchestrator**  
  Takes the Judge’s scores and chooses **final trade direction**:
  - Long, short, or flat.
  - Engines themselves do **not** size risk or stack positions.

- **Risk & Pyramids**  
  Risk is handled **after** direction is chosen:
  - Position sizing, filters, and portfolio constraints live in the risk layer.
  - A **pyramid module** scales into winners according to configurable rules.

---

## Engine Families

Mammon currently defines five engine families:

### 1. MACD – MomentumOS

MACD-based momentum system (this repo’s first dev focus).

- Uses MACD histogram, slopes, and velocity to score trend strength.
- Separate long/short logic with two “angles” per side (e.g., velocity vs convergence).
- Each sub-engine:
  - Emits a score / signal only.
  - Has its own kill switch and cooldown.
  - Flags its state (live / killed) to the Judge.

Typical use: core momentum backbone that runs on most symbols/timeframes.

---

### 2. Levels – Pivot

Price-level & pivot-based engine family.

- Works around key levels: pivots, ranges, levels of interest.
- Two angles per side (for example: mean-reversion vs breakout at levels).
- Long/short engines share the same level map but have independent logic & kills.

Typical use: structure-aware entries around defined price levels.

---

### 3. JetStream – Volume-Weighted Bands + VWAP

JetStream combines **volume-weighted Bollinger Bands** with **VWAP logic**.

- Bands are adjusted by volume context rather than just price volatility.
- VWAP is used as a directional and “gravity” reference.
- Two angles per side (e.g., trend-following vs fade around volume bands).
- Kill switches protect against regime shifts and failed mean-reversions.

Typical use: trend & pullback logic with explicit volume context.

---

### 4. WhipSaw – Volatility-Weighted Bands + VWAP

WhipSaw focuses on **volatility-weighted Bollinger Bands** plus the **same VWAP
framework** as JetStream, but tuned for chop and mean-reversion.

- Bands expand/contract based on volatility to detect whipsaw zones.
- VWAP is reused as a bias / equilibrium line.
- Two angles per side (fade vs breakout inside noisy ranges).
- Per-engine kill logic to shut down when chop turns into directional trend.

Typical use: extracting edge from choppy regimes instead of getting shredded by them.

---

### 5. Turtle – Breakout Engine

Turtle is a breakout-style engine family.

- Classic channel / range breakouts adapted to the Mammon scoring model.
- Two angles per side (fast / slow or short-term / long-term breakouts).
- Long and short breakouts run independently with their own kills.

Typical use: catching large moves and trend initiations.

---

## Judge & Orchestrator

### Judge

The **Judge** is a meta-layer that:

- Reads all engine scores and on/off flags.
- Aggregates them into **composite long and short scores**.
- Optionally weights engines differently by family, timeframe, or regime.

It does **not** place trades directly; it just hands the orchestrator a clean view
of where the system wants to be.

### Orchestrator

The **Orchestrator**:

- Compares long vs short composite scores.
- Chooses **final side** (long, short, or flat).
- Triggers the final **entry/exit signals** that the strategy uses.

Engines are “pure signal.” The orchestrator is where those get turned into a
single, tradeable decision.

---

## Risk & Pyramiding

Direction first, risk second:

- Risk module runs **after** the orchestrator chooses long/short.
- Handles:
  - Position sizing
  - Filters (max exposure, volatility caps, etc.)
  - Trade frequency or cooldown logic
- A dedicated **pyramid function** scales into winning trades according to
  configured steps and thresholds.

Engines don’t know about risk or stacking; they just vote. Risk decides how hard
to press the bet.

---

## Status

This repository is being built out in stages:

- `MACD / MomentumOS` engines and labs.
- Then additional engine families (Levels, JetStream, WhipSaw, Turtle).
- Then Judge, Orchestrator, Risk, and Pyramid modules.

The goal is a clean, modular codebase that can be lifted into other platforms or
rewritten in other languages while preserving the same engine/Judge architecture.
