# Lightweight Dialogue Policy Adapters for Full‑Duplex Multimodal LLMs (ALPS 2026 mini project)
**Date:** 2026-03-23  
**Format:** 1-day mini project (6 people) → concrete research plan 

## 1) One-paragraph project summary
This project explores **parameter-efficient “dialogue policy adapters” (LoRA-like)** to improve **turn-taking** for existing **full‑duplex audio LLM systems** (e.g., Moshi; SALMONN‑omni as candidates described in literature). Rather than building turn-taking from scratch, we aim to learn a lightweight adaptation layer that improves *when the agent speaks, waits, or pauses*— improving perceived conversational naturalness and responsiveness—using **small conversational datasets**.

## 2) Scope 
- **Audio-only** (no video cues)
- Turn-taking behaviors:
- **(A)** decides when the agent should **start speaking while it’s silent**;
- **(B)** decides whether/when the agent should **stop (yield) when the user starts speaking during the agent’s speech**. 
- **Adapters / LoRA-like** fine-tuning as the intended parameter-efficient method 

## 3) Why this matters
Full‑duplex audio agents can be improved on interaction mechanics. Improving turn-taking:
- increases perceived **naturalness**
- improves **responsiveness/latency** (both timing and reaction)
- enables domain/style norms (customer support, coaching, etc.) with minimal retraining

## 4) Candidate bases to evaluate 
This project includes identifying the best base target by:
- availability (weights/code access)
- license and usage constraints
- ease of running locally/offline
- existence of streaming/full‑duplex  controls
- feasibility of LoRA/adapters in the stack

Initial thoughts:
- Moshi: https://github.com/kyutai-labs/moshi
- SALMONN‑omni: https://arxiv.org/abs/2505.17060
- NVIDIA PersonaPlex (policy/persona inspiration): https://research.nvidia.com/labs/adlr/personaplex/

## 5) Proposed approach
### Key idea
Add a lightweight learned component that outputs **turn-taking control signals** conditioned on:
- recent audio context (user/agent speech activity)
- conversational state features (who has the floor, ...)
- optional text/semantic context (if available) 

### Policy interface (to be extended/improved)
Define a minimal action space to be implemented:
- `SPEAK`  — start/continue agent speech generation
- `WAIT`   — hold response; do not start speaking
- `YIELD`  — immediately stop (or pause) the agent’s current speech output when the user starts talking

*(Later extensions: `INTERRUPT`, `BACKCHANNEL`, etc.)*

### Where adapters/LoRA fit 
Here is an initial design option:
1. **Token/control-first**: LoRA adapts the base model to emit special control tokens interpreted by the runtime.
The project will also explicitly evaluate alternative approaches.

## 6) Data (what’s needed)
### Minimum training signal needed for training a turn-taking system
To train or imitate turn-taking offline we need conversational audio with:
- **who spoke when** (speaker diarization segments)
- **overlap regions** (simultaneous speech)
- associated timestamps 

### Data acquisition paths (to be evaluated in the project)
- Public conversational audio corpora with overlap / multi-speaker properties
- Public meeting datasets (often have overlap; diarization may be available or derivable)
- Synthetic overlap construction (mix two single-speaker streams to simulate barge-in events) for toy training

### Minimal annotation pipeline (offline)
- VAD (voice activity detection) → speech segments
- Diarization (speaker segmentation) → speaker turns
- Derive labels:
  - “should speak now?” based on floor ownership heuristics
  - “should yield?” when user starts speaking during agent speech


## 7) 1-day project deliverables (expected concrete outcomes)
### Deliverable A — Model selection memo + decision matrix
- 2–3 best candidates + runner-up(s)
- explicit pros/cons: availability, license, LoRA support, full-duplex hooks, ease of use
- recommended “first target” for a follow-on implementation

### Deliverable B — Data + annotation plan
- 1–2 public dataset candidates for toy training
- minimal pipeline steps (VAD/diarization) with tooling suggestions

### Deliverable C — Toy offline training evidence (tiny demo)
- train a minimal policy classifier/regressor predicting `SPEAK` vs `WAIT/YIELD` from derived features
- baseline: heuristic policy (e.g., simple VAD floor rule)

### Deliverable D — Communication 
- Slides outline + narrative
- (Optionally) Video/demo concept storyboard


## 8) Recommended reads / prep
- Moshi: https://github.com/kyutai-labs/moshi
- SALMONN‑omni: https://arxiv.org/abs/2505.17060
- PersonaPlex: https://research.nvidia.com/labs/adlr/personaplex/

