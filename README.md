# Lightweight Dialogue Policy Adapters for Full‑Duplex Multimodal LLMs (ALPS 2026 mini project)
**Date:** 2026-03-23  
**Format:** 1-day feasibility sprint (6 people) → concrete plan + tiny offline training evidence

## 1) One-paragraph project summary
This project explores **parameter-efficient “dialogue policy adapters” (LoRA-like)** to improve **turn-taking** for existing **full‑duplex audio LLM systems** (e.g., Moshi; SALMONN‑omni as a candidate described in literature). Rather than building turn-taking from scratch, we aim to learn a lightweight adaptation layer that improves *when the agent speaks, waits, or yields*—reducing collisions, handling barge-ins more gracefully, and improving perceived conversational naturalness and responsiveness—using **small conversational datasets** and offline evaluation harnesses.

## 2) Scope (v1)
### In scope (v1)
- **Audio-only** (no video cues)
- Turn-taking behaviors:
  - **(A) When to speak vs. wait** (floor control)
  - **(B) Interruptions / yielding** (barge-in handling)
- **Adapters / LoRA-like** fine-tuning as the intended parameter-efficient method (final implementation target)
- **Offline analysis + offline toy training** (1-day sprint constraint)

### Out of scope (v1)
- Video-based cues (gaze, facial expressions)
- Rich prosody control / expressive TTS controls (can be future work)
- Full production deployment; real-time demo is not required in the 1-day sprint

## 3) Why this matters
Full‑duplex audio agents often fail on interaction mechanics: they talk over users, hesitate too long, or ignore barge-ins. Improving turn-taking:
- increases perceived **naturalness**
- improves **responsiveness/latency** (both timing and reaction)
- enables domain/style norms (customer support, coaching, etc.) with minimal retraining

## 4) Candidate bases to evaluate (feasibility-driven)
This sprint includes identifying the best base target by:
- availability (weights/code access)
- license and usage constraints
- ease of running locally/offline
- existence of streaming/full‑duplex hooks and “stop talking” controls
- feasibility of LoRA/adapters in the stack

Initial references:
- Moshi: https://github.com/kyutai-labs/moshi
- SALMONN‑omni: https://arxiv.org/abs/2505.17060
- NVIDIA PersonaPlex (policy/persona inspiration): https://research.nvidia.com/labs/adlr/personaplex/

## 5) Proposed approach (conceptual architecture)
### Key idea
Add a lightweight learned component that outputs **turn-taking control signals** conditioned on:
- recent audio context (user/agent speech activity)
- conversational state features (who has the floor, overlap indicators, recent yields)
- optional text/semantic context if available (not required for v1 feasibility)

### Policy interface (conceptual)
Define a minimal action space to be implemented by a runtime controller:
- `SPEAK`  — start/continue agent speech generation
- `WAIT`   — hold response; do not start speaking
- `YIELD`  — stop or pause agent speech upon user barge-in

*(Later extensions: `INTERRUPT`, `BACKCHANNEL`, domain-specific norms.)*

### Where adapters/LoRA fit (design choices to decide later)
The sprint output should explicitly evaluate these options:
1. **Controller-first**: a small policy network (possibly with LoRA on an encoder/LLM) decides actions; base model handles content.
2. **Token/control-first**: LoRA adapts the base model to emit special control tokens interpreted by the runtime.
3. **Hybrid**: controller gates streaming + LoRA nudges content generation under timing constraints.

## 6) Data plan (what’s needed)
### Minimum training signal needed for v1 turn-taking
To train or imitate turn-taking offline we need audio with:
- **who spoke when** (speaker diarization segments)
- **overlap regions** (simultaneous speech)
- timestamps sufficient to compute turn gaps and barge-in timing

### Data acquisition paths (to be evaluated in the sprint)
- Public conversational audio corpora with overlap / multi-speaker properties
- Public meeting datasets (often have overlap; diarization may be available or derivable)
- Synthetic overlap construction (mix two single-speaker streams to simulate barge-in events) for toy training

### Minimal annotation pipeline (offline)
- VAD (voice activity detection) → speech segments
- Diarization (speaker segmentation) → speaker turns
- Derive labels:
  - “should speak now?” based on floor ownership heuristics
  - “should yield?” when user starts speaking during agent speech

## 7) Evaluation plan (offline, feasibility-friendly)
### Proxy metrics for “naturalness”
- **Overlap/collision rate**: fraction of time agent and user speak simultaneously
- **Turn transition gap distribution**: too-long gaps vs too-short gaps
- **Yield reaction time proxy**: time from user speech onset during overlap → agent stop

### Latency proxies (offline)
- End-of-turn response timing: user speech end → predicted agent start
- Responsiveness during barge-in: user speech start while agent speaking → predicted yield

### Qualitative outputs (for the end-of-day video concept)
- Timeline visualizations: waveform + diarization bars + predicted actions
- Side-by-side comparison: baseline heuristic vs learned policy

## 8) 1-day sprint deliverables (expected concrete outcomes)
### Deliverable A — Model selection memo + decision matrix
- 2–3 best candidates + runner-up(s)
- explicit pros/cons: availability, license, LoRA support, full-duplex hooks, ease of use
- recommended “first target” for a follow-on implementation

### Deliverable B — Architecture spec
- chosen v1 policy interface (`SPEAK/WAIT/YIELD`)
- required runtime hooks for a full-duplex system (start/stop speaking, barge-in handling)
- training and inference diagram (even if implementation deferred)

### Deliverable C — Data + annotation plan
- 1–2 public dataset candidates for toy training
- minimal pipeline steps (VAD/diarization) with tooling suggestions
- what “small data” means operationally (e.g., number of minutes/conversations)

### Deliverable D — Toy offline training evidence (tiny demo)
- train a minimal policy classifier/regressor predicting `SPEAK` vs `WAIT/YIELD` from derived features
- baseline: heuristic policy (e.g., simple VAD floor rule)
- report at least one measurable improvement on proxy metrics

### Deliverable E — Communication artifacts
- Slides outline + narrative
- Video/demo concept storyboard:
  - “offline replay UI” showing predicted actions over diarization
  - baseline vs adapted policy comparison

## 9) Suggested team split (6 people, parallel)
1. **Model survey lead**: Moshi + SALMONN‑omni feasibility, LoRA hooks, licenses, runtime control points
2. **Policy/adapter design lead**: define action space, controller interface, LoRA insertion hypotheses
3. **Data lead**: identify dataset candidates; confirm download and suitability for overlap/diarization
4. **Annotation pipeline lead**: quick VAD/diarization workflow; produce labels for toy training
5. **Toy training lead**: implement minimal classifier/regressor + baseline heuristic; compute proxy metrics
6. **Comms lead**: draft final narrative, slide outline, demo storyboard; consolidate outputs

## 10) Recommended reads / prep
- Moshi: https://github.com/kyutai-labs/moshi
- SALMONN‑omni: https://arxiv.org/abs/2505.17060
- PersonaPlex: https://research.nvidia.com/labs/adlr/personaplex/

Prep checklist (before sprint starts):
- Align on v1 actions: `SPEAK / WAIT / YIELD`
- Ensure at least one public dataset is selected and downloadable
- Decide baseline heuristic for comparison
- Decide reporting format: one table of metrics + 2–3 timeline plots
