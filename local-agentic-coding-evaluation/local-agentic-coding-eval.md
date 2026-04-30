# Field Notes: Local LLM as a Coding Agent — A Practical Evaluation

> What happens when you skip the API, strap a 24GB GPU to your desk, and let a local model write your application from scratch? These are the unfiltered notes.

---

## Why This Exists

<!-- TODO: 2-3 sentences. Frame the motivation:
     - You wanted to evaluate whether local LLMs are viable as agentic coders
     - The caption-engine app is a byproduct — a real, working application that fell out of the eval
     - Link to repo: https://github.com/cubebecu/caption-engine
-->

This document is a field report, not a tutorial. It covers what worked, what didn't, and what surprised me while building a complete application using only a locally-hosted LLM as the coding agent. The resulting app — [caption-engine](https://github.com/cubebecu/caption-engine) — is functional, Dockerized, and solves a real problem. It also happens to be the clearest proof that this workflow can produce shipping software.

---

## Hardware

### The GPU: NVIDIA RTX 3090

<!-- TODO: Fill in your actual specs and acquisition details -->

| Spec | Value |
|------|-------|
| VRAM | 24 GB GDDR6X |
| Memory bandwidth | 936 GB/s |
| CUDA cores | 10,496 |
| Architecture | Ampere (GA102) |
| TDP | 350W |
| Acquired | <!-- TODO: used/refurbished? price paid? --> |

### GPU Condition & Refurbishment

This card was acquired used and required hands-on refurbishment before it could sustain the 100% GPU utilization that agentic coding demands for extended sessions. The original thermal paste had calcified, causing throttling under sustained inference loads. After disassembly, cleaning, and repasting, core temperatures dropped by approximately 30°C.

Full hardware writeup with photos, diagnostics, and before/after thermals: [Bringing a Used RTX 3090 Back to Life](<!-- TODO: link to hardware writeup -->)

---

## Software Stack

### Inference Server

<!-- TODO: Describe your setup:
     - llama.cpp / llama-server as the inference backend
     - How you compiled it (CUDA flags, quantization support)
     - How you exposed the API (port, OpenAI-compatible endpoint)
     - Any custom parameters (batch size, thread count, etc.)
-->

### Agentic Coding Tool

<!-- TODO: Which agentic coder did you connect to the local endpoint?
     - Tool name and version
     - How you pointed it at the local API instead of a hosted one
     - Any configuration changes required
     - Temperature, sampling settings if relevant
-->

### Model Configuration

<!-- TODO: Specific model details for the CODING agent (not the caption app's Gemma):

| Parameter | Value |
|-----------|-------|
| Model | <!-- e.g. Qwen2.5-Coder-32B --> |
| Quantization | <!-- e.g. Q4_K_M --> |
| Context window | <!-- tokens --> |
| KV cache quantization | <!-- Q8_0 / Q4_0 / none --> |
| Effective VRAM usage | <!-- GB --> |

     Explain calibration decisions — why this quant level, why this context 
     size, what tradeoffs you accepted. This is the part most writeups skip 
     and most readers actually need. -->

---

## Model Evaluation

### Model A: Qwen MoE — The Disaster

<!-- TODO: Which specific MoE model? Size? Quantization?
     
     Document the failure modes with specifics:
     - Infinite code generation — did it hit context limits or just not stop?
     - Hallucinated APIs / libraries that don't exist
     - Looping — repeating the same blocks?
     - Code that looked plausible but didn't run?
     - How many iterations before you abandoned it?
     
     Hypothesis on WHY (this is the valuable part):
     - MoE routing degradation under quantization?
     - Expert activation patterns unsuitable for sustained code generation?
     - Context window handling differences vs. dense?
     - "I suspect X but haven't confirmed" is valid and worth stating. -->

### Model B: Qwen Dense — The Workhorse

<!-- TODO: Which specific dense model? Size? Quantization?
     
     Document what worked:
     - Code quality — what percentage went to production without edits?
     - Multi-file awareness
     - Framework comprehension (FastAPI, Docker, HTML/JS frontend)
     - Where it still needed human correction
     - Average prompt iterations per feature
     
     Name what the LLM actually built — be concrete:
     - FastAPI backend with health monitoring and circuit breaker
     - llama.cpp integration with GPU layer offloading
     - Web UI with 4 tabs (caption, results, logs, configuration)
     - Job persistence with thumbnails
     - Image validation (size, format, dimension limits)
     - Docker deployment with compose
     - That's not a toy project. Say so. -->

### Side-by-Side

<!-- TODO: 

| Dimension | MoE | Dense |
|-----------|-----|-------|
| Completion rate | | |
| Code quality | | |
| Hallucination frequency | | |
| Context coherence | | |
| Iterations per feature | | |
| Verdict | ❌ Unusable | ✅ Production-viable |
-->

---

## Unscientific but Honest Metrics

<!-- TODO: THE signature section.

     Profanity count during local LLM sessions vs. your baseline with 
     Claude Code. Present it straight — developer frustration is a real cost 
     and standard metrics don't capture it.

     Then make it useful — categorize what CAUSED the frustration:
     - Hallucinated APIs or nonexistent methods?
     - Ignoring explicit instructions?
     - Context window amnesia mid-task?
     - Slow generation speed?
     - Correct logic, wrong file / wrong location?
     - Something else?
     
     This turns a joke metric into an actual frustration taxonomy.
     
     Optional: include the screenshot showing 9M tokens processed. -->

---

## Cost Analysis

### Per-Session Economics

<!-- TODO: You have hard data — 9M tokens in one session.

| | Local (RTX 3090) | Anthropic Sonnet | Anthropic Opus |
|---|---|---|---|
| Tokens processed | 9M | 9M | 9M |
| API cost | — | $X.XX | $Y.YY |
| Electricity cost | ~$Z (350W × hours × local rate) | — | — |

### Total Cost of Ownership — Honest Version

| Component | Cost |
|-----------|------|
| GPU acquisition | €/$ XXX |
| Electricity (this eval) | €/$ X.XX |
| Setup & configuration time | X hours |
| Sessions to break even vs. Sonnet API | ~N |
| Sessions to break even vs. Opus API | ~N |

     The point isn't "local is always cheaper." The point is showing real 
     numbers so someone else can run their own calculation. If you don't 
     include TCO, someone will call it out — and they'd be right. -->

---

## The Result

<!-- TODO: 
     - Animated GIF of the app workflow (upload → generate → markdown output)
     - Brief description: what caption-engine does and why it exists
     - Emphasize the scope: this is a Dockerized service with health checks,
       circuit breakers, API endpoints, job persistence, input validation —
       not a weekend script.
     - Link: https://github.com/cubebecu/caption-engine
-->

![caption-engine demo](<!-- TODO: path/to/demo.gif -->)

[**→ caption-engine repository**](https://github.com/cubebecu/caption-engine)

---

## Conclusions

Local LLMs as coding agents are a viable option today — with clear boundaries.

**Where it works:**
<!-- TODO: Expand each with your actual observations -->
- Small to mid-size greenfield applications with well-defined scope
- Contained tasks with clear boundaries and short feedback loops
- Environments where data sensitivity or cost rules out hosted APIs
- A credible Plan B for teams dependent on API availability

**Where it doesn't (yet):**
<!-- TODO: Same — ground each in what you observed -->
- Deep reasoning requiring multi-step logical chains
- Large-scale refactoring across established codebases
- Tasks that exceed the context window (and most real projects do)
- Sustained long-context work where the model needs to track many files

**The main limiter is context window.** <!-- TODO: This is your key finding.
     What specifically breaks? Does the model:
     - Forget earlier instructions?
     - Contradict its own code from 20 messages ago?
     - Lose track of file structure?
     - Start re-implementing things it already wrote?
     Be concrete — this is what people will quote. -->

---

## Limitations of This Evaluation

This eval covers one scenario: a single greenfield application, built from scratch, by one developer. It does not test debugging legacy code, multi-developer workflows, large multi-file refactoring, or extended projects where context accumulates over weeks. Two model variants were tested; results may differ substantially with other architectures, sizes, or quantization methods.

Treat these findings as n=1 field data, not benchmarks.

---

## About

<!-- TODO: The "middle voice" — neither expert nor amateur.
     Something like: "Solutions engineer who builds things to understand them —
     from repasting GPUs to training nanoGPT, from Kubernetes clusters 
     to ESP32 firmware." 
     Link to GitHub profile and/or LinkedIn. -->

---

*Published: <!-- TODO: date -->*  
*Last updated: <!-- TODO: date -->*  
*v1.0 — will update as models and hardware evolve.*
